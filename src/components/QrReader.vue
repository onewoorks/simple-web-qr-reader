<template>
  <div>
    <qrcode-stream @decode="onDecode" @init="onInit" />
    <div>
      <div>{{ customer.message }}</div>
      <table class="table table-bordered">
        <tbody>
          <tr>
            <th>Name</th>
            <td>{{ customer.name }}</td>
            <td rowspan="4" style="vertical-align:middle">
              Sitting Position
              <div>{{ customer.sitting_zone }}</div>
            </td>
          </tr>
          <tr>
            <th>Email</th>
            <td>{{ customer.email }}</td>
          </tr>
          <tr>
            <th>Phone Number</th>
            <td>{{ customer.phone_number }}</td>
          </tr>
          <tr>
            <th>Register Date</th>
            <td>{{ customer.register_date }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="d-flex">
      <button type="button" class="btn btn-primary btn-block" @click="onConfirm">Confirm</button>
      <button type="button" class="btn btn-default">Cancel</button>
    </div>
  </div>
</template>

<script>
import Axios from "axios";
import { QrcodeStream } from "vue-qrcode-reader";

export default {
  name: "QrReader",
  components: {
    QrcodeStream
  },
  data: function() {
    return {
      customer: {}
    };
  },
  methods: {
    onDecode(result) {
      this.reader = result;
    },
    onConfirm() {
      var customer_data = this.customer
      var post_data  = {
        "customer_name"     : customer_data.name,
        "customer_phone_no" : customer_data.phone_number,
        "email"             : customer_data.email,
        "register_date"     : customer_data.register_date,
        "reader_counter"    : "1",
        "sitting_zone"      : customer_data.sitting_zone,
        "id_number"         : customer_data.id_number.toString(),
        "qr_code"           : customer_data.id_number.toString()
      }
      Axios.post(process.env.VUE_APP_API+'/attendence/attend',post_data)
    },
    async onInit(promise) {
      try {
        await promise;
      } catch (error) {
        if (error.name === "NotAllowedError") {
          this.error = "ERROR: you need to grant camera access permisson";
        } else if (error.name === "NotFoundError") {
          this.error = "ERROR: no camera on this device";
        } else if (error.name === "NotSupportedError") {
          this.error = "ERROR: secure context required (HTTPS, localhost)";
        } else if (error.name === "NotReadableError") {
          this.error = "ERROR: is the camera already in use?";
        } else if (error.name === "OverconstrainedError") {
          this.error = "ERROR: installed cameras are not suitable";
        } else if (error.name === "StreamApiNotSupportedError") {
          this.error = "ERROR: Stream API is not supported in this browser";
        }
      }
    }
  },
  mounted: function() {
    Axios.get(process.env.VUE_APP_API+"/attendence/find/4").then(response => {
      var data = response.data;
      this.customer = {
        id_number     : data.detail.id,
        message       : data.message,
        name          : data.detail.name,
        phone_number  : data.detail.phone_number,
        email         : data.detail.email,
        register_date : data.detail.register_date,
        sitting_zone  : data.detail.sitting_zone
      };
    });
  }
};
</script>


<style>
.btn {
  border-radius: 0;
}
</style>