<template>
  <div :class="{ 'fullscreen': fullscreen }" 
      :key="componentKey"
      ref="wrapper" @fullscreenchange="onFullscreenChange">
        <div class="qrreader">
          <img src="img/logo.png" alt="nsummit logo" style='height:80px' /> 
        </div>
    <qrcode-stream @decode="onDecode" @init="onInit" />
  <a 
    href="#myModal" 
    role="button" 
    class="d-none btn btn-primary" 
    data-backdrop="static" data-keyboard="false"
    data-toggle="modal" 
    ref="ticketModal">Launch modal</a>
    <div class="modal" id="myModal" tabindex="-1" role="dialog" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered" role="document">
        <div class="modal-content">
            <div class="modal-header model-header-success text-center">
                <h5 class="modal-title">SITTING ZONE : {{ customer.sitting_zone }}</h5>
            </div>
            <div class="modal-body p-4" id="result">
                    <div style='font-weight:bold'>{{ customer.name }}</div>
                    <div>{{ customer.phone_number }}</div>
                    <div>{{ customer.email }}</div>
                    <div>{{ customer.register_date }}</div>
            </div>
            <div class="modal-footer">
                  <button 
                    v-if="customer.valid == 1" 
                    type="button" 
                    class='btn btn-warning btn-block'
                    @click="onClose">{{ customer.message }}</button>
                  <button v-else type="button" @click="onConfirm" class="btn btn-block btn-success" data-dismiss="modal">CONFIRM</button>
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
      customer: {},
      fullscreen: true,
      componentKey: 0,
    };
  },
  methods: {
    onFullscreenChange(event) {
      this.fullscreen = document.fullscreenElement !== null
    },

    requestFullscreen() {
      const elem = this.$refs.wrapper

      if (elem.requestFullscreen) {
        elem.requestFullscreen();
      } else if (elem.mozRequestFullScreen) { /* Firefox */
        elem.mozRequestFullScreen();
      } else if (elem.webkitRequestFullscreen) { /* Chrome, Safari and Opera */
        elem.webkitRequestFullscreen();
      } else if (elem.msRequestFullscreen) { /* IE/Edge */
        elem.msRequestFullscreen();
      }
    },

    exitFullscreen() {
      if (document.exitFullscreen) {
        document.exitFullscreen();
      } else if (document.mozCancelFullScreen) { /* Firefox */
        document.mozCancelFullScreen();
      } else if (document.webkitExitFullscreen) { /* Chrome, Safari and Opera */
        document.webkitExitFullscreen();
      } else if (document.msExitFullscreen) { /* IE/Edge */
        document.msExitFullscreen();
      }
    },

    logErrors (promise) {
      promise.catch(console.error)
    },
    onDecode(result) {
      this.reader = result;
      this.getResult(result)
    },
    onClose(){
      this.forceRerender()
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
        "qr_code"           : customer_data.qr_code.toString()
      }
      Axios.post(process.env.VUE_APP_API+'/attendence/attend',post_data)
      this.forceRerender()
    },
    forceRerender() {
      this.componentKey += 1;  
    },
    getResult(read){
      Axios.get(process.env.VUE_APP_API+`/attendence/find/${read}`)
        .then(response => {
          var data = response.data;
          var manual = (data.code == 950 ) ? true : false 
          this.customer = { 
            id_number     : (manual) ? "" : data.detail.id,
            qr_code       : (manual) ? "" : data.detail.code,
            message       : data.message,
            name          : (manual) ? "" : data.detail.name.toUpperCase(),
            phone_number  : (manual) ? "" : data.detail.phone,
            email         : (manual) ? "" : data.detail.email,
            register_date : (manual) ? "" : data.detail.register_date,
            sitting_zone  : (manual) ? "ERROR!!!" : data.detail.sitting,
            valid         : (manual) ? 1 : data.detail.attend_status
          };
          this.$refs.ticketModal.click()
      });
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

  }
};
</script>


<style scoped>
.btn {
  border-radius: 0;
}
.modal-dialog-centered {
  align-items: flex-end
}

.fullscreen {
  position: fixed;
  z-index: 1000;
  top: 0;
  bottom: 0;
  right: 0;
  left: 0;
}

.fullscreen-button {
  background-color: white;
  position: absolute;
  bottom: 0;
  right: 0;
  margin: 1rem;
}
.fullscreen-button img {
  width: 2rem;
}
.qrreader {
  position: absolute;
  z-index: 1001;
  left: 10px;
  top:10px
}
</style>