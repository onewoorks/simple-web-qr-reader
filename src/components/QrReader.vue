<template>
    <div>
        <qrcode-stream @decode="onDecode" @init="onInit" />
        <div>{{ reader }}</div>
        <div>
            <table class="table table-bordered">
                <tbody>
                    <tr>
                        <th>Name</th>
                        <td>{{ customer.name }}</td>
                        <td rowspan="4" style="vertical-align:middle">
                            
                            Sitting Position
                        </td>
                    </tr>
                    <tr>
                        <th>Email</th>
                        <td></td>
                    </tr>
                    <tr>
                        <th>Phone Number</th>
                        <td></td>
                    </tr>
                    <tr>
                        <th>Register Date</th>
                        <td></td>
                    </tr>
                </tbody>
            </table>
        </div>

            
        
        <div class="d-flex">
            <button type="button" class="btn btn-primary btn-block">Confirm</button>
            <button type="button" class="btn btn-default">Cancel</button>
        </div>
    </div>
</template>

<script>

import { QrcodeStream } from 'vue-qrcode-reader'

export default {
    name : "QrReader",
    components: { 
        QrcodeStream
        },
    data: function() {
        return { 
            reader: "qr result",
            customer: {}
        }
    },
    methods: {
    onDecode (result) {
      this.reader = result
    },
    async onInit (promise) {
      try {
        await promise
      } catch (error) {
        if (error.name === 'NotAllowedError') {
          this.error = "ERROR: you need to grant camera access permisson"
        } else if (error.name === 'NotFoundError') {
          this.error = "ERROR: no camera on this device"
        } else if (error.name === 'NotSupportedError') {
          this.error = "ERROR: secure context required (HTTPS, localhost)"
        } else if (error.name === 'NotReadableError') {
          this.error = "ERROR: is the camera already in use?"
        } else if (error.name === 'OverconstrainedError') {
          this.error = "ERROR: installed cameras are not suitable"
        } else if (error.name === 'StreamApiNotSupportedError') {
          this.error = "ERROR: Stream API is not supported in this browser"
        }
      }
    }
    },
    mounted: function(){
        this.customer = {
            name: "IRWAN IBRAHIM"
        }
    }
}
</script>


<style>
.btn {
    border-radius: 0
}
</style>