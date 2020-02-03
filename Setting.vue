<template>
  <div class="profile_data_main">
    <div class="row setting-page">
      <div class="col-xl-10 col-sm-12 mb-9">
        <h5 class="page-header-title">Update Settings</h5>
          <flash-message :flash="flash" @close="alert = false" v-show="alert"></flash-message>
          <h6 class="page-header-title" v-if="isAuthAdmin">Site Setting</h6>
          <form @submit.prevent="updateSettings">
            <div class="site-setting" v-if="isAuthAdmin">
          <div class="form-inline row" >
            <div class="col-md-2">
              <label for="title" class="control-label">Title </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left"><img :src="config.baseUrl+'images/name.png'"></span>
                </div>
                <input required type="text" name="site_title" class="form-control left" placeholder="Site Title" v-model="site.title">
              </div>
            </div>
          </div>
          <br>
          <div class="form-inline row">
            <div class="col-md-2">
              <label for="email" class="control-label">Email </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left"><img :src="config.baseUrl+'images/envelop.png'"></span>
                </div>
                <input required type="email" name="site_email" class="form-control left" placeholder="Site Email" v-model="site.email">
              </div>
            </div>
          </div>
          <br>
          <div class="form-inline row">
            <div class="col-md-2">
              <label for="commission" class="control-label">Commission </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left control-label align-middle">
                    <i class="fa fa-percent"></i>
                  </span>
                </div>
                <input required type="text" name="commission" class="form-control left" placeholder="Web-Smarter Commission" v-model="site.commission" v-on:keydown="$options.filters.decimalAllowed($event)">
              </div>
            </div>
          </div>
          <br>
          <div class="form-inline row">
            <div class="col-md-2">
              <label for="task_response_duration" class="control-label">Task Response Duration </label>
            </div>
            <div class="col-md-8">
              <div class="input-group modified-select2">
                <select2 :options="duration_type.options" v-if="site.task_response.type.length>0" v-model="site.task_response.type" cls="form-control col-md-2" placeholder="Select Duration Type" v-on:input="">
                </select2>
                <div class="input-group-prepend  ml-2">
                  <span class="input-group-text left control-label align-middle">
                    <i class="fa fa-clock-o fa-lg" aria-hidden="true"></i>
                  </span>
                </div>
                <input required type="text" name="task_response_duration" v-model="site.task_response.time" class="form-control col-md-10 left" placeholder="Duration" v-on:keydown="$options.filters.numberAllowed($event)">                
              </div>
            </div>
          </div>
          <br>
          <br>
          <h6 class="page-header-title">Client Setting</h6>
          <fieldset class="setting-area">
          <legend><h5>Default-Setting</h5></legend>
           <div class="form-inline row">
            <div class="col-md-2">
              <label for="client_rate" class="control-label">Rate:</label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <!-- <span class="input-group-text left"><img :src="config.baseUrl+'images/client-rate.png'"></span> -->
                  <span class="input-group-text left control-label align-middle"><i class="fa fa-dollar"></i></span>
                </div>
                <input required type="text" name="client_rate" v-model="site.client.rate" class="form-control left" placeholder="Rate" v-on:keydown="$options.filters.decimalAllowed($event)">
              </div>
            </div>
          </div>
          <br>
          <div class="form-inline row">
            <div class="col-md-2">
              <label for="client_amount" class="control-label">Amount:</label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left control-label align-middle"><i class="fa fa-dollar"></i></span>
                </div>
                <input type="text" name="client_amount" v-model="site.client.amount" class="form-control left" placeholder="Amount" v-on:keydown="$options.filters.decimalAllowed($event)">
              </div>
            </div>
          </div>
          </fieldset>
          <br>
          <br>
        </div>
        
        <div class=""  v-if="isAuthAgency">
          <div class="form-inline row">
            <div class="col-md-2">
              <label for="client_rate" class="control-label">Client Rate:</label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <!-- <span class="input-group-text left"><img :src="config.baseUrl+'images/client-rate.png'"></span> -->
                  <span class="input-group-text left control-label align-middle"><i class="fa fa-dollar"></i></span>
                </div>
                <input required type="text" name="client_rate" id="client_rate" v-model="site.client.rate" class="form-control left" placeholder="Rate" v-on:keydown="$options.filters.decimalAllowed($event)">
              </div>
            </div>
          </div>
          <br>
        </div>
        <!-- <h6 class="page-header-title">PAY</h6> -->
        <fieldset class="setting-area">
        <legend><h5>PAY</h5></legend>
        <div class="form-inline row">
          <div class="col-md-2">
            <label class="control-label">To Worker</label>
          </div>
          <div class="col-md-8">
            <div class="input-group">
              <span>
                <input type="radio" v-model="pay.worker" value="1" id="worker_by_web_smarter" class="d-inline align-middle">
                <label for="worker_by_web_smarter" class="control-label d-inline">By Web-smarter</label>
              </span>
              <span class="ml-4">
                <input type="radio" v-model="pay.worker" value="0" id="worker_externally_web_smarter" class="d-inline align-middle">
                <label for="worker_externally_web_smarter" class="control-label d-inline">Externally Web-smarter</label>
              </span>
            </div>
          </div>
        </div>
        </fieldset>
        <br>
        <br>
        <div class="paypal" v-if="isAuthAdmin">
          <br>
          <h6 class="page-header-title">Paypal Setting</h6>
          <fieldset class="setting-area">
          <legend><h5>User</h5></legend>
          <div class="form-inline row">
            <div class="col-md-2">
              <label class="control-label" for="sandox_user">Sandbox: </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left">
                    <img :src="config.baseUrl+'images/user.png'">
                  </span>
                </div>
                <input type="text" v-model="paypal.user.sandbox" name="sandox_user" class="form-control left" placeholder="User">
              </div>
            </div>
          </div>
          <br>
          <div class="form-inline row">
            <div class="col-md-2">
              <label class="control-label" for="production_user">Production: </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left">
                    <img :src="config.baseUrl+'images/user.png'"></span>
                </div>
                <input type="text" v-model="paypal.user.production" name="production_user" class="form-control left" placeholder="User">
              </div>
            </div>
          </div>
          </fieldset>
          <br>
          <fieldset class="setting-area">
          <legend><h5>Password</h5></legend>
          <div class="form-inline row">
            <div class="col-md-2">
              <label class="control-label" for="sandox_password">Sandbox: </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left">
                    <img :src="config.baseUrl+'images/lock.png'">
                  </span>
                </div>
                <input type="text" v-model="paypal.password.sandbox" name="sandox_password" class="form-control left" placeholder="Password">
              </div>
            </div>
          </div>
          <br>
          <div class="form-inline row">
            <div class="col-md-2">
              <label class="control-label" for="production_password">Production: </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left">
                    <img :src="config.baseUrl+'images/lock.png'"></span>
                </div>
                <input type="text" v-model="paypal.password.production" name="production_password" class="form-control left" placeholder="Password">
              </div>
            </div>
          </div>
          </fieldset>
          <br>
          <fieldset class="setting-area">
          <legend><h5>Signature</h5></legend>
          <div class="form-inline row">
            <div class="col-md-2">
              <label class="control-label" for="sandbox_signature">Sandbox: </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left">
                    <img :src="config.baseUrl+'images/signature.png'"></span>
                </div>
                <input type="text" v-model="paypal.signature.sandbox" name="sandbox_signature" class="form-control left" placeholder="Signature">
              </div>
            </div>
          </div>
          <br>
          <div class="form-inline row">
            <div class="col-md-2">
              <label class="control-label" for="production_signature">Production: </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left">
                    <img :src="config.baseUrl+'images/signature.png'"></span>
                </div>
                <input type="text" v-model="paypal.signature.production" name="production_signature" class="form-control left" placeholder="Signature">
              </div>
            </div>
          </div>
          </fieldset>
          <br>
          <fieldset class="setting-area">
          <legend><h5>Client-ID</h5></legend>
          <div class="form-inline row">
            <div class="col-md-2">
              <label class="control-label" for="sandox_client_id">Sandbox: </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left">
                    <img :src="config.baseUrl+'images/signature.png'">
                  </span>
                </div>
                <input type="text" name="sandox_client_id" v-model="paypal.client_id.sandbox" class="form-control left" required placeholder="SandBox Client-ID">
              </div>
            </div>
          </div>
          <br>
          <div class="form-inline row">
            <div class="col-md-2">
              <label class="control-label" for="production_client_id">Production: </label>
            </div>
            <div class="col-md-8">
              <div class="input-group">
                <div class="input-group-prepend">
                  <span class="input-group-text left">
                    <img :src="config.baseUrl+'images/signature.png'">
                  </span>
                </div>
                <input type="text" name="production_client_id" v-model="paypal.client_id.production" class="form-control left" required placeholder="Production Client-ID">
              </div>
            </div>
          </div>
          </fieldset>
          <br>
          <div class="form-inline row">
            <div class="col-md-2">
              <label for="paypal_mode" class="control-label">Mode </label>
            </div>
            <div class="col-md-8">
              <div class="input-group modified-select2" v-if="paypal.mode.length>0">
                <select2 :options="paypal_mode" v-model="paypal.mode" v-on:input="updateMode" cls="form-control" placeholder="Select Paypal Mode">
                </select2>
              </div>
            </div>
          </div>
          <br>
          </div>
            <br>
            <div class="form-inline row">
              <div class="col-md-2"></div>
              <div class="col-md-8 text-left">
                <button type="submit" class="btn btn-warning">Update</button>
              </div>
            </div>
            <br><br>
          </form>
        </div>
    </div>   
  </div>
</template>
<script>
    export default {
        props: [],
        data(){
            return{
              old:{
                commission:0
              },
              paypal:{
                user:{
                  sandbox:"",
                  production:""
                },
                password:{
                  sandbox:"",
                  production:""
                },
                signature:{
                  sandbox:"",
                  production:""
                },
                client_id:{
                  sandbox:"",
                  production:""
                },
                mode:''
              },
              site:{
                client:{rate:0,amount:0},
                email:'',
                title:'',
                commission:0,
                task_response:{time:0,type:''}
              },
              flash:{
                error: false,
                message : ""                 
              },
              pay:{
                worker:1,
                agency:0
              },
              paypal_mode:[],
              duration_type:[],
              alert:false,
              auth:this.$auth.user()               
            }
        },
        created: function()
        { 
          document.title = this.$route.meta.title;
          this.getSiteSettings();
        },
        computed:{
          isAuthAdmin(){
            return this.auth.role_id==0?true:false;
          },
          isAuthAgency(){
            return this.auth.role_id==3?true:false;
          }
        },
        methods: {
          getSiteSettings(){
              this.$emit('loaderShow');
              let uri = '/site_settings/'+this.auth.email;
              this.axios.get(uri).then((response) => {
                this.paypal = response.data.paypal;
                this.site = response.data.site;
                this.paypal_mode = response.data.paypal_mode;
                this.duration_type = response.data.durationType;
                this.pay = response.data.pay;
                if(!this.site.task_response){
                  this.site = $.extend(true,this.site,{task_response:{time:0,type:this.duration_type.default}})
                }
                this.$emit('loaderHide');
              });
            
          },
          updateSettings(){
            this.$emit('loaderShow');
            let uri = '/site_settings/'+this.auth.email;          

            this.site.client.rate = this.$options.filters.formatPrice(this.site.client.rate);
            this.site.client.amount = this.$options.filters.formatPrice(this.site.client.amount);

            this.axios.post(uri,{paypal:this.paypal,site:this.site,pay:this.pay}).then((response) => {
              this.$emit('loaderHide');
              this.alert = true;
              if(response.data.code==0){
                this.flash.error = true;
              }
              this.flash.message = response.data.message;
            });
            this.$options.filters.flashHide(this);
          },          
          updateMode(val){
            if(val){
            }
          }
        }
    }
</script>
