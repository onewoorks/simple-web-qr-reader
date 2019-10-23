<template>
  <div>
    <qrcode-stream @decode="onDecode" @init="onInit" />
    <div class="d-none">
      <div>{{ customer.message }}</div>
      <table aria-describedby="" class="table table-bordered">
        <tbody>
          <tr>
            <th scope="col">Name</th>
            <td>{{ customer.name }}</td>
            <td rowspan="4" style="vertical-align:middle">
              Sitting Position
              <div>{{ customer.sitting_zone }}</div>
            </td>
          </tr>
          <tr>
            <th scope="col">Email</th>
            <td>{{ customer.email }}</td>
          </tr>
          <tr>
            <th scope="col">Phone Number</th>
            <td>{{ customer.phone_number }}</td>
          </tr>
          <tr>
            <th scope="col">Register Date</th>
            <td>{{ customer.register_date }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="d-flex hide">
      <button type="button" class="btn btn-primary btn-block d-none" @click="onConfirm">Confirm</button>
      <button type="button" class="btn btn-default d-none">Cancel</button>
    </div>

    <lottie-animation
                      path="https://assets7.lottiefiles.com/packages/lf20_Vwcw5D.json"
                      :loop="false"
                      :autoPlay="true"
                      :loopDelayMin="2.5"
                      :loopDelayMax="5"
                      :speed="1"
                      :width="256"
                      :height="256"
                  />

  <a href="#myModal" role="button" class="d-none btn btn-primary" data-toggle="modal" ref="ticketModal">Launch modal</a>
    <div class="modal" id="myModal" tabindex="-1" role="dialog" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered" role="document">
        <div class="modal-content">
            <div class="modal-header model-header-success text-center">
                <h5 class="modal-title">INFORMATION</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">×</span>
                </button>
            </div>
            <div class="modal-body p-4" id="result">
                    
                    <div>
                      Attendee Information
                    </div>
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-block btn-success" data-dismiss="modal">CONFIRM</button>
            </div>
        </div>
    </div>
</div>  
  </div>

  
</template>

<script>
import Axios from "axios";
import { QrcodeStream } from "vue-qrcode-reader";
import LottieAnimation from '../../node_modules/lottie-vuejs/src/LottieAnimation'

export default {
  name: "QrReader",
  components: {
    QrcodeStream,
    LottieAnimation
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
    // Axios.get(process.env.VUE_APP_API+"/attendence/find/4")
    //   .then(response => {
    //     var data = response.data;
    //     this.customer = { 
    //       id_number     : data.detail.id,
    //       message       : data.message,
    //       name          : data.detail.name,
    //       phone_number  : data.detail.phone_number,
    //       email         : data.detail.email,
    //       register_date : data.detail.register_date,
    //       sitting_zone  : data.detail.sitting_zone
    //     };
    // });
    this.$refs.ticketModal.click()
  }
};
</script>


<style>
.btn {
  border-radius: 0;
}
.modal-dialog-centered {
  align-items: flex-end
}
</style>