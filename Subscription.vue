<template>
  <div class="profile_data_main">
    <div class="row setting-page">
      <div class="col-xl-10 col-sm-12 mb-9">
        <h5 class="page-header-title">Subscriptions</h5>
          <flash-message :flash="flash" @close="alert = false" v-show="alert"></flash-message>
          <form @submit.prevent="updateSettings">
            <div class="site-setting" v-if="isAuthAdmin">
          <div class="form-inline row" >
            <div class="col-md-2">
              <label for="period" class="control-label">Billing Period  </label>
            </div>
            <div class="col-md-8">
              <div class="input-group modified-select2">
                <div class="input-group-prepend">
                  <span class="input-group-text left control-label align-middle"><i aria-hidden="true" class="fa fa-clock-o fa-lg"></i></span>
                </div>
                <select2 :options="periodOptions" cls="form-control left col-md-10" placeholder="Select Period" v-model="formData.period" v-if="periodOptions">
                  <option value="" disabled="">Select Period</option>
                </select2>
              </div>
            </div>
          </div>
          <br>
          <div class=""  v-for="(plan,key) in formData.plans">
          <div class="form-inline row">
            <div class="col-md-2"  v-if="key==0">
              <label for="amount" class="control-label">Plan Amount </label>
            </div>
            <div class="col-md-8" :class="key>0?'offset-2':''">
              <div class="input-group" >
                <div class="input-group-prepend">
                  <span class="input-group-text left control-label align-middle"><i class="fa fa-dollar"></i></span>
                </div>
                <input required type="text" v-model="plan.amount" class="form-control col-md-10 left" placeholder="Amount" v-on:keydown="$options.filters.numberAllowed($event);" v-on:keyup="checkValue($event,key)"> 
                <a class="align-middle m-auto btn" v-if="key==0" @click.prevent="addPlan()"><i class="fa fa-lg fa-plus"></i></a>
                <a class="align-middle m-auto btn" v-if="key>0" @click.pervent="deletePlan(key)"><i class="fa fa-fw fa-trash-o"></i></a>
              </div>
            </div>
          </div>
          <br></div>
        </div>
            <div class="form-inline row">
              <div class="col-md-2"></div>
              <div class="col-md-8 text-left">
                <button type="button" class="btn btn-warning" @click="clearAll()">Clear All</button>
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
              formData:{
                period:'',
                bonus:'',
                plans:[{amount:0}]
              },
              periodOptions:null,
              alert:false,
              flash:{
                error: false,
                message : ""                 
              },
              auth:this.$auth.user(),
              old:{bonus:'',plans:[{amount:0}]},           
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
              let uri = '/site_settings/'+this.auth.email+'?subscribe=true';          
              this.axios.get(uri).then((response) => {
                if(response.data.subscriptions.plans){
                  this.formData = response.data.subscriptions;
                  this.old.plans = this.formData.plans;
                }
                this.periodOptions = response.data.periodOptions;
                this.$emit('loaderHide');
              });
          },
          updateSettings(){

            this.$emit('loaderShow');
            let uri = '/site_settings/'+this.auth.email;          

            this.axios.post(uri,{subscription:this.formData}).then((response) => {
              this.$emit('loaderHide');
              this.alert = true;
              if(response.data.code==0){
                this.flash.error = true;
              }
              this.flash.message = response.data.message;
            });
            this.$options.filters.flashHide(this);
          },          
          addPlan(){
            this.formData.plans.push({amount: 0});
            this.old.plans = this.formData.plans;
          },
          deletePlan(index){
            this.formData.plans.splice(index,1);
            this.old.plans = this.formData.plans;
          },
          clearAll(){
            this.formData = {period:'',bonus:0,plans:[{amount:0}]};
            this.old.plans = this.formData.plans;
          },
          checkValue(event,index){
          }
        }
    }
</script>
