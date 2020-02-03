<template>
  <div class="profile_data_main">
    <div class="row setting-page">
      <div class="col-xl-10 col-sm-12 mb-9">
        <h5 class="page-header-title">Bonus</h5>
          <flash-message :flash="flash" @close="alert = false" v-show="alert"></flash-message>
          <form @submit.prevent="updateSettings">
          <div class="site-setting" v-if="isAuthAdmin">
          <div class="form-inline row">
            <div class="col-md-8  offset-2 text-center">
              <div class="input-group" >
                <div class="col-md-3 col-xs-3 p-0">
                  <b>From</b>
                </div>
                &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                <div class="col-md-3 col-xs-3 p-0">
                  <b>To</b>
                </div>
                &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                <div class="col-md-3 col-xs-3 p-0">
                  <b>Bonus</b>
                </div>
              </div>
            </div>
          </div>
          <br>
          <div class=""  v-for="(plan,key) in formData">
          <div class="form-inline row">
            <div class="col-md-8 offset-2">
              <div class="input-group" >
                <div class="input-group-prepend">
                  <span class="input-group-text left control-label align-middle"><i class="fa fa-dollar"></i></span>
                </div>
                <input required type="text" v-model="plan.from" class="form-control col-md-2 left" placeholder="Amount" v-on:keydown="$options.filters.numberAllowed($event);" v-on:keyup="old[key].from=$event.target.value;"  v-on:blur="checkValue($event,key)"> 
                &nbsp;&nbsp;&nbsp;&nbsp;
                <div class="input-group-prepend">
                  <span class="input-group-text left control-label align-middle"><i class="fa fa-dollar"></i></span>
                </div>
                <input required type="text" v-model="plan.to" class="form-control col-md-2 left" placeholder="Amount" v-on:keydown="$options.filters.numberAllowed($event);" v-on:keyup="old[key].to=$event.target.value;" v-on:blur="checkValue($event,key,'to')"> 
                &nbsp;&nbsp;&nbsp;&nbsp;
                <div class="input-group-prepend">
                  <span class="input-group-text left control-label align-middle"><i class="fa fa-percent"></i></span>
                </div>
                <input required type="text" v-model="plan.bonus" min="0" max="100" class="form-control col-md-2 left" placeholder="Bonus" v-on:keydown="$options.filters.decimalAllowed($event);" v-on:keyup="old[key].bonus = $options.filters.minMax($event,old[key].bonus); formData[key].bonus=old[key].bonus; checkValue($event,key)"> 
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
              formData:[{
                from:0,
                to:0,
                bonus:0
              }],
              periodOptions:null,
              alert:false,
              flash:{
                error: false,
                message : ""                 
              },
              auth:this.$auth.user(),
              old:[{bonus:'',from:'',to:''}],           
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
              let uri = '/site_settings/'+this.auth.email+'?bonus=true';          
              this.axios.get(uri).then((response) => {
                if(response.data.bonus.length>0){
                  this.formData = response.data.bonus;
                  this.old = this.formData;
                }
                this.$emit('loaderHide');
              });
          },
          updateSettings(){
            this.$emit('loaderShow');
            let uri = '/site_settings/'+this.auth.email;          

            console.log(this.formData);
            this.axios.post(uri,{bonus:this.formData}).then((response) => {
              this.$emit('loaderHide');
              this.alert = true;
              if(response.data.code==0){
                this.flash.error = true;
              }
              this.flash.message = response.data.message;
            });
            //this.$options.filters.flashHide(this);
          },          
          addPlan(){
            if(this.checkNotEmpty()){
              this.formData.push({bonus: 0,to: 0,from: 0});
              this.old = this.formData;
            }
          },
          deletePlan(index){
            this.formData.splice(index,1);
            this.old = this.formData;
          },
          clearAll(){
            this.formData = [{from:'',to:'',bonus:0}];
            this.old = this.formData;
          },
          checkNotEmpty(){
            let error = false;
            for(var i=0; i < this.formData.length; i++){
              if(i!=0 && this.formData[i].from==0 ){
               error = true;
               //return false; 
              }
              if(this.formData[i].to==0){
                error = true;
                //return false;
              }
            }
            if(error){
              this.alert = true;
              this.flash.error = true;
              this.flash.message = "Please fill inputs first";
              return false;
            }else{
              return true;
            }
          },
          checkValue(event,index,type='from'){
          }
        }
    }
</script>
