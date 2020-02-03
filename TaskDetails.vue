<template>
    <div class="profile_data_main">
        <div class="" v-if="isAdmin()">
            <h5 class="page-header-title eps_heading pull-left">Task Details</h5>
            <div class="right-head-top-worker pull-right">
                <router-link :to="{ name: 'tasks' }">
                    <img :src="config.baseUrl+'images/back_task.png'" class="img-responsive" width="180">
                </router-link>
            </div>
            <div class="table-responsive">
                <table class="table tbl-task-detail">
                    <tr>
                        <td>Task Id</td>
                        <td>#{{ task.id }}</td>
                    </tr>
                    <tr>
                        <td>Client Name</td>
                        <td>{{ task.client_name }}</td>
                    </tr>
                    <tr>
                        <td>Task Name</td>
                        <td>{{ task.name }}</td>
                    </tr>
                    <tr>
                        <td>Url</td>
                        <td><a :href="task.url" target="_blank">{{ task.url }}</a></td>
                    </tr>
                    <tr>
                        <td>Description</td>
                        <td v-html="task.message"></td>
                    </tr>
                    <tr v-show="task.image">
                        <td>Screenshot</td>
                        <td><img :src="task.image" width="500" class="img-screenshot"></td>
                    </tr>
                    <tr>
                        <td>Status</td>
                        <td>{{ task.status_name }}</td>
                    </tr>
                    <tr v-show="task.worker_name">
                        <td>Assigned Worker</td>
                        <td>{{ task.worker_name }}</td>
                    </tr>
                    <tr v-show="task.estimation">
                        <td>Estimated Hour</td>
                        <td>{{ task.estimation }}</td>
                    </tr>
                    <tr v-show="task.amount">
                        <td>Amount</td>
                        <td>{{ task.amount }}</td>
                    </tr>
                </table>
            </div>
            <div class="pending_request worker-list" v-if="task.status>=3 && ((isAdmin() && task.team=='web') || isAgency)">
                <div class="row mb-3">
                    <div class="col-md-5"><h5 class="page-header-title m-auto">Workers assigned to the task</h5></div>
                    <div class="col-md-7 align-right" v-if="task.status==3 && noWorkerLeft">
                        <a href="javascript:void(0);" @click.stop.prevent="showOpenTaskModal=true;" class="btn btn-primary eps_btn" v-if="isAgency && task.isTaskOpen==0">
                            <span class="plus_sign">
                                <img :src="config.baseUrl+'images/tick.png'" class="img-responsive" width="22">
                            </span> Open Task
                        </a>
                        <a href="javascript:void(0);" @click.stop.prevent="showAssignModal=true;" class="btn btn-primary eps_btn" v-if="task.isTaskOpen==0">
                            <span class="plus_sign">
                                <img :src="config.baseUrl+'images/plus.png'" class="img-responsive" width="22">
                            </span> Assign Workers
                        </a>
                    </div>
                    <br>
                </div>
                <div class="row">
                  <div class="col-md-12">
                    <div class="bgc-white bd">
                       <div class="table-responsive">
                      <table class="table">
                        <thead>
                          <tr>
                            <th scope="col" width="30%">Name</th>
                            <th scope="col" width="30%">Email</th>
                            <th scope="col" width="28%">Estimation</th>
                            <th scope="col" width="12%">Status</th>
                          </tr>
                        </thead>
                        <tbody>
                          <tr class="manage_user_list" v-for="worker,key in workersList">
                              <td>{{ worker.name }}</td>
                              <td>{{ worker.email }}</td>
                              <td>
                                <span v-if="worker.estimation!=null">
                                    {{ estimatedTime(key) }}
                                </span>
                              </td>
                              <td>
                                <div class="status-wrapper btn-success" v-if="worker.active==1">Active</div>
                                <div class="status-wrapper status-in-progress" v-else-if="worker.estimation!=null && worker.leaved==0">Estimated</div>
                                <div class="status-wrapper status-ready" v-else-if="worker.assign==1 && worker.leaved==0">Assign</div>
                                <div class="status-wrapper btn-danger" v-else="worker.leaved==1">Leaved</div>
                              </td>
                          </tr>
                          <tr v-if="workersList.length==0">
                            <td colspan="4" class="text-center">No record found</td>
                          </tr>
                        </tbody>
                        <tfoot>
                          <tr>
                            <td colspan="4">      
                              <div class="text-center">
                                <div class="paginate-count pagination pull-left">Total Records : {{ workersList.length }}</div>
                              </div>
                            </td>
                          </tr>
                        </tfoot>
                      </table>
                    </div>
                    </div>
                  </div>
                </div>
            </div>
            <div>
                <model-popup v-if="showAssignModal" :class="showAssignModal?'show':''" @close="newWorkers=[]; showAssignModal = false">
                  <h3 slot="header">New Workers</h3>
                  <div slot="body">
                    <form v-on:submit.prevent="assignNewWorkerOnTask()">
                      <div class="modal-body modified-select2">
                          <select2 :options="newWorkersList" cls="form-control col-md-12" v-model="newWorkers" multiple="true" data-placeholder="Select Worker">
                            <option disabled value="0">Select Worker</option>
                          </select2>
                      </div>
                      <div class="modal-footer">
                          <button class="btn btn-primary eps_btn" type="submit">Assign</button>
                          <button class="btn btn-danger eps_btn" @click.prevent="newWorkers=[]; showAssignModal = false">No</button>
                      </div>
                      </form>
                  </div>
                </model-popup>
                <model-popup v-if="showOpenTaskModal" :class="showOpenTaskModal?'show':''" @close="newWorkers=[]; showOpenTaskModal = false">
                  <h3 slot="header">Open Task</h3>
                  <div slot="body">
                    <form v-on:submit.prevent="openTask()">
                      <div class="modal-body modified-select2">
                          Are you sure to open this task for web-smarter?
                      </div>
                      <div class="modal-footer">
                          <button class="btn btn-primary eps_btn" type="submit">Yes</button>
                          <button class="btn btn-danger eps_btn" @click.prevent="showOpenTaskModal = false">No</button>
                      </div>
                      </form>
                  </div>
                </model-popup>
            </div>
        </div>
        <div v-else>
            <div class="row">
                <div class="col-xl-12 col-sm-12 mb-9">
                    <div class="box box-custom">
                        <div class="main-display-table">
                            <h5 class="page-header-title eps_heading pull-left">
                                Task # {{ task.id }} 
                                <span class="status-wrapper ml-3 worker-header-back" :class="getClass(task.status)"> 
                                    {{ task.status_name }}
                                </span>
                            </h5>
                            <div class="right-head-top-worker pull-right">
                                <router-link :to="{ name: 'tasks' }">
                                    <img :src="config.baseUrl+'images/back_task.png'" class="img-responsive" width="180">
                                </router-link>
                            </div>
                        </div>

                        <div class="box-body">
                            <h3>{{ task.name }}
                                <div class="pull-right tasks-list" v-if="isClient()">
                                    <ul class="action-list">
                                        <li v-if="task.status<3">
                                            <a class="status-estimation btn" v-on:click.stop.prevent="askForEstimation()">
                                                <span class="wps_paypal_icon">
                                                    <img :src="config.baseUrl+'images/calc.png'">
                                                </span>
                                                Ask for Estimation
                                            </a>
                                        </li>
                                    
                                        <li class="mark-status" v-if="task.status==6">
                                            <a v-on:click.stop.prevent="updateStatus('reopen')" class="status-re-open btn" data-action="Reopen" style="margin-left:5px; ">
                                                <span class="wps_paypal_icon">
                                                    <img :src="config.baseUrl+'images/search.png'">
                                                </span>
                                                Reopen the Task
                                            </a>
                                        </li>
                                        <li class="mark-status" v-if="task.status==6">
                                            <a v-on:click.stop.prevent="updateStatus('done')" class="status-done btn" data-action="Complete" style="margin-left:5px; ">
                                                <span class="wps_paypal_icon">
                                                    <img :src="config.baseUrl+'images/tick.png'">
                                                </span>
                                                Mark Task as Done
                                            </a>
                                        </li>
                                    
                                    </ul>
                                </div>
                                
                                <div class="pull-right tasks-list" v-if="!isClient() && taskAssigned && (task.status==5 || task.status==7) && task.task_detail_id==details.id">
                                    <ul class="action-list">
                                        <li>
                                            <a class="status-review btn" v-on:click.stop.prevent="updateStatus('review')" id="send-review">
                                                <span class="wps_paypal_icon">
                                                    <i class="fa fa-search" aria-hidden="true"></i>
                                                </span>
                                                Send to Review
                                            </a>
                                        </li>
                                    </ul>
                                </div>
                                <div class="pull-right tasks-list" v-if="isWorker && (task.status>=3 && task.status<=4)">
                                    <ul class="action-list">
                                        <li>
                                            <button class="btn btn-info" @click="showModal = true">Estimation</button>
                                        </li>
                                        <li v-if="!task.estimation">
                                            <button class="btn btn-info" @click="showLeaveModal = true">Leave Task</button>
                                        </li>
                                    </ul>
                                </div>
                            </h3>
                            <ul>
                                <li>Created : {{ task.created }}</li>
                                <li>Last Modified : {{ task.modified }}</li>
                                <li v-if="task.amount">Amount : {{ task.amount }} </li>
                                <li v-if="task.estimation">Hours : {{ task.estimation }}</li>
                                <li v-if="isClient && task.worker_name">Assigned Worker : {{ task.worker_name }}</li>
                                <li v-if="!isClient">Client Name : {{ task.client_name }}</li>
                                <li>URL : <a :href="task.url" class="" target="_blank">{{task.url}}</a></li>
                            </ul>
                            <br>

                            <div class="nav-tabs-custom">
                                <ul class="nav nav-tabs" id="inviteTabs">
                                    <li :class="tabDiscussion" @click="tabActive('message')"><a href="#tab_message" data-toggle="tab" aria-expanded="true">Task Discussion</a></li>
                                    <li :class="tabWorker" @click="tabActive('worker')" v-if="isClient()"><a href="#tab_worker" data-toggle="tab" aria-expanded="true">Workers</a></li>          
                                </ul>
                                <div class="tab-content">
                                    <div class="tab-pane" :class="tabDiscussion" id="tab_message">
                                        <div class="direct-chat-messages">
                                            <div class="direct-chat-msg" v-if="threads.length>0" v-for="(item,key) in threads">
                                                <div class="img_section">
                                                    <img alt="image" class="img-circle direct-chat-img" :src="config.baseUrl+'images/default-user.png'" width="50px" height="50px"> 
                                                </div>
                                                <div class="msg_section">
                                                    <div class="direct-chat-info pull-left">
                                                        <span class="direct-chat-name pull-left">{{item.user_id==auth.id?'You':item.user.name}}</span>
                                                        <span class="direct-chat-timestamp pull-left"><i class="fa fa-clock-o" aria-hidden="true"></i> {{ item.created_at }}</span>
                                                    </div>
                                                <br>
                                                    <div class="direct-chat-text pull-left">
                                                        <p v-html="item.message"></p>
                                                        <br>
                                                        <div class="chat_img_wrapper clear" v-if="task.image && key==0">
                                                            <img :src="task.image" alt="discuss">
                                                        </div>
                                                    </div>

                                                </div>
                                            </div>
                                        </div>
                                        <div class="row" v-if="task.status!=8">
                                            <div class="col-sm-1">
                                                <img width="50px" height="50px" :src="config.baseUrl+'images/default-user.png'" class="img-circle direct-chat-img" alt="image">
                                            </div>
                                            <div class="col-sm-11 div-msg-input">
                                                <div class="input-group">
                                                    <input class="form-control right" placeholder="Say something..." v-model="message" name="message" type="text" @keyup.enter="sendMessage">
                                                    <div class="input-group-prepend">
                                                        <button type="submit" overlay="true" class="add-overlay btn-send" @click="sendMessage">
                                                            <img alt="image" class="" :src="config.baseUrl+'images/send.png'">
                                                        </button>
                                                    </div>
                                                </div>     
                                            </div>
                                        </div>

                                    </div>
                                    <div class="tab-pane" id="tab_worker" :class="tabWorker" v-if="isClient()">
                                        <table class="table table-striped">
                                            <thead>
                                                <tr>
                                                    <th scope="col" width="3%">#</th>
                                                    <th scope="col" width="40%">Worker Name</th>
                                                    <th scope="col" width="30%">Amount</th>
                                                    <th scope="col"></th>
                                                    <th  width="12%"></th>
                                                </tr>
                                            </thead>
                                            <tbody>
                                                <tr v-for="(worker,key) in details">
                                                    <th scope="row">{{key + 1}}</th>
                                                    <td>
                                                        {{worker.uname}}
                                                    </td>
                                                    <td>
                                                        {{ currency+' '+ (worker.estimation*task.rate).toFixed(2) }}
                                                    </td>              
                                                    <td>
                                                        {{ worker.isAssigned==1?'Assigned':''}}
                                                    </td>              
                                                    <td>                                                    
                                                    </td>
                                                </tr>
                                                <tr v-if="details==null" >
                                                    <td colspan="5" class="text-center">No record found</td>
                                                </tr>
                                            </tbody>
                                        </table>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            <div v-if="isWorker">
                <model-popup v-if="showModal" :class="showModal?'show':''" @close="showModal = false">
                <!--
                  you can use custom content here to overwrite
                  default content
                -->
                    <h3 slot="header">Estimation</h3>
                    <div slot="body">
                        <form v-on:submit.prevent="giveEstimation()">
                            <div class="modal-body">
                                <div class="col-md-12 text-center estimation-popup">
                                    <div class="col-md-4 d-inline-block">
                                        <input type="text" class="form-control text-center" v-model="estimation.hour" v-on:keydown="$options.filters.numberAllowed($event)" placeholder="Hours">
                                        <span class="time">Hours</span>
                                    </div> 
                                    <span class="col-md-1 mt-1 d-inline-block align-top">
                                        <i aria-hidden="true" class="fa fa-ellipsis-v mt-2"></i>
                                    </span> 
                                    <div class="col-md-4 d-inline-block">
                                        <input type="text" class="form-control text-center" v-model="estimation.min" v-on:keydown="$options.filters.numberAllowed($event)" placeholder="Minutes">
                                        <span class="time">Minutes</span>
                                    </div> 
                                    <!-- <span class="col-md-1 mt-1 d-inline-block align-top">
                                        <i aria-hidden="true" class="fa fa-ellipsis-v mt-2"></i>
                                    </span> 
                                    <div class="col-md-3 d-inline-block">
                                        <input type="text" class="form-control text-center" v-model="estimation.sec" v-on:keydown="$options.filters.numberAllowed($event)" placeholder="Seconds">
                                        <span class="time">Seconds</span>
                                    </div> -->
                                </div>
                            </div>
                            <div class="modal-footer">
                                <button class="btn btn-primary">Update</button>
                                <button class="btn btn-danger" @click.prevent="showModal = false">Close</button>
                            </div>
                        </form>
                    </div>
                </model-popup>
                <model-popup v-if="showLeaveModal" :class="showLeaveModal?'show':''" @close="showLeaveModal = false">
                  <h3 slot="header">Leave Task</h3>
                  <div slot="body">
                      <div class="modal-body modified-select2">
                          Are you sure to leave this task?
                      </div>
                      <div class="modal-body">
                          <button class="btn btn-primary" v-on:click.prevent="leaveTask()">Yes</button>
                          <button class="btn btn-danger" @click.prevent="showLeaveModal = false">No</button>
                      </div>
                  </div>
              </model-popup>
            </div>
            <div v-if="isClient()">
                <model-popup v-if="showEstimationModal" :class="showEstimationModal?'show':''" @close="showEstimationModal = false">
                  <h3 slot="header">Move Task Under Estimation</h3>
                  <div slot="body">
                      <form v-on:submit.prevent="askForEstimation()">
                          <div class="modal-body modified-select2">
                              <select2 :options="teamOptions" cls="form-control col-md-12" v-model="defaultTeam" data-placeholder="Select Team">
                                <option disabled value="0">Select Team</option>
                              </select2>
                          </div>
                          <div class="modal-footer">
                              <button class="btn btn-primary">Update</button>
                              <button class="btn btn-danger" @click.prevent="showEstimationModal = false">Close</button>
                          </div>
                      </form>
                  </div>
              </model-popup>
            </div>
        </div>
    </div>
</template>

<script>

    //var modal = require('../common/ModelPopUp.vue');

    //If you want to use any component only on particular page then we need to import on that page and pass in component attribute in 
    //import ModalPopUp from '../common/ModelPopUp.vue'
    
    export default{
        props: ['id'],//'config',
        /*components: {
            'modal': ModalPopUp
        },*/
        data(){
            return{
                estimation:{hour:0,min:0},
                task:{},
                details:{},
                threads:{},
                clientShow:false,
                tabDiscussion:'active',
                tabWorker:'',
                taskAssigned:false,
                currency:'',
                showOpenTaskModal:false,
                showAssignModal:false,
                showModal: false,
                showLeaveModal:false,
                showEstimationModal:false,
                noWorkerLeft:false,
                teamOptions:[],
                defaultTeam:'web',
                message:'',
                workersList:[],
                newWorkersList:[],
                newWorkers:[],
                auth:this.$auth.user()
            }
        },
        created: function(){
            document.title = this.$route.meta.title;
            this.$emit('loaderHide');
            this.getItem();
        },
        mounted(){
           
        },
        computed: {   
            isWorker: function () {
              return this.auth.role_id==2?true:false;
            },
            isAgency: function () {
              return this.auth.role_id==3?true:false;
            },           
        },
        methods: {
            isAdmin: function () {
              return this.auth.role_id==0 || this.auth.role_id==3?true:false;
            },
            isClient: function () {
              return this.auth.role_id==1?true:false;
            },  
            estimatedTime(index) {
                if(this.workersList[index].estimation !== null && this.workersList[index].estimation !== ''){
                    let es = this.workersList[index].estimation.split('.');
                    let hr = (es[0]>0?es[0]+' Hrs.':'');
                    let min = (es.length>1 && es[1]>0?parseInt(es[1]*60/100):'');
                    if(min>0){
                        min = (min.length==1?min+'0':min)+' Min.';
                    }

                    return hr+(es.length>1 && min.length>0?' '+min:min);
                    //return this.workersList[index].estimation;
                }
            },   
            tabActive(tab){
                if(tab=='message'){
                    this.tabDiscussion = 'active';
                    this.tabWorker = '';
                }else{
                    this.tabDiscussion = '';
                    this.tabWorker = 'active';
                }
            },    
            getClass(sId){
                let className = '';
                switch (sId) {
                    case 3:
                        className = 'status-estimation';
                        break;
                    case 4:
                        className = 'status-ready';
                        break;
                    case 5:
                        className = 'status-in-progress';
                        break;
                    case 6:
                        className = 'status-review';
                        break;
                    case 7:
                        className = 'status-re-open';
                        break;
                    case 8:
                        className = 'status-done';
                        break;
                    default:
                        className = 'status-drafts';
                }
                return className;
            },
            getItem()
            {
                this.$emit('loaderShow');
                let uri = '';
                uri = '/task/'+this.auth.email+'/'+this.$route.params.id;
                this.axios.get(uri).then((response) => {
                    if(response.data.code==1){
                        this.task = response.data.task;
                        
                        if((this.isAdmin() && this.task.team=='web') || this.isAgency){
                            this.getWorkers();    
                        }
                        
                        this.threads = response.data.threads;
                        this.taskAssigned = response.data.taskAssigned;
                        this.details = response.data.details;
                        this.currency = response.data.currency;
                        if(!this.isAdmin() && !this.isClient()){
                            if(this.details!=null){
                                this.estimation = this.task.estimated_time;
                            }
                        }
                        if(this.auth.parent_id && response.data.team){
                            this.teamOptions = response.data.team.withAgency;
                        }else if(response.data.team){
                            this.teamOptions = response.data.team.withOutAgency;
                        }
                    }else{
                        this.$router.replace({ name: 'tasks'});
                    }
                    this.$emit('loaderHide');
                });
                
            },
            sendMessage(){
                //alert("test"+this.message);
                if(this.message){
                    this.$emit('loaderShow');
                    let uri = '/task/send_message/'+this.auth.email+'/'+this.$route.params.id;
                    this.axios.post(uri,{"message":this.message} ).then(response => {
                        this.$emit('loaderHide');
                        this.getItem();
                    })
                    this.message = "";
                }
            },
            updateStatus(ustatus){
                this.$emit('loaderShow');
                let uri = '/task/update_status/'+this.auth.email+'/'+this.task.id+'/'+ustatus;
                this.axios.get(uri).then((response) => {
                  if(response.data.code==1){
                    this.task.status_name = response.data.status_name;
                    this.task.status = response.data.status;
                  }
                  this.$emit('loaderHide');
                });
            },
            callEstimationPopup(){
                this.showEstimationModal = true;
            },
            askForEstimation(){
                let uri = '/task/ask_estimation/'+this.auth.email+'/'+this.task.id;
                this.$emit('loaderShow');
                this.axios.post(uri,{team:this.defaultTeam}).then((response) => {
                  if(response.data.code==1){
                    this.task.status_name = response.data.status_name;
                    this.task.status = response.data.status;
                  }
                  this.$emit('loaderHide');
                });
            },
            giveEstimation(){
                let uri = '/task/update_estimation/'+this.auth.email+'/'+this.task.id;
                if(this.estimation.hour>0 || this.estimation.min>0){
                    this.axios.post(uri,{'estimation':this.estimation}).then((response) => {
                      if(response.data.code==1){
                        this.showModal = false;
                        this.task.estimation = (this.estimation.hour>0?this.estimation.hour+' Hour ':'')+(this.estimation.min>0?this.estimation.min+' Min.':'');
                        this.task.amount = response.data.amount; 
                        if(response.data.status_name){
                            this.task.status_name = response.data.status_name;
                            this.task.status = response.data.status;
                        }                    
                      }
                    });
                }
            },
            leaveTask(){
                let uri = '/task_leave/'+this.auth.email+'/'+this.task.id;
                this.$emit('loaderShow');
                this.axios.post(uri,{leave:true}).then((response) => {
                    if(response.data.code==1){
                        this.showLeaveModal = false;
                        this.$router.replace({ name: 'tasks'});    
                    }
                    this.$emit('loaderHide');
                });
                
            },
            getWorkers(){
                let uri = '/task_workers/'+this.auth.email+'/'+this.$route.params.id;
                this.$emit('loaderShow');
                this.axios.get(uri).then((response) => {                    
                    if(response.data.code==1){
                        this.workersList = response.data.workersList;
                        this.noWorkerLeft = response.data.noWorkerLeft;
                        this.newWorkersList = response.data.newWorkersList;
                    }
                    this.$emit('loaderHide');
                });
            },
            isNumber: function() {
                let evt = window.event;
                var charCode = (evt.which) ? evt.which : evt.keyCode;
                if ((charCode > 31 && (charCode < 48 || charCode > 57)) && charCode !== 46) {
                    evt.preventDefault();;
                } else {
                    return true;
                }
            },
            assignNewWorkerOnTask(){
                if(this.newWorkers.length>0){
                    let uri = '/assign_new_task_worker/'+this.auth.email+'/'+this.$route.params.id;
                    this.$emit('loaderShow');
                    this.axios.post(uri,{workers:this.newWorkers}).then((response) => {                    
                        if(response.data.code==1){
                            this.showAssignModal = false;
                            this.newWorkers = [];
                            this.getWorkers();
                        }
                        this.$emit('loaderHide');
                    });
                }else{
                    this.showAssignModal = false;
                }                
            },
            openTask(){
                if(this.noWorkerLeft){
                    let uri = '/task_open/'+this.auth.email+'/'+this.$route.params.id;
                    this.$emit('loaderShow');
                    this.axios.post(uri,{open:true}).then((response) => {                    
                        this.showOpenTaskModal = false;
                        if(response.data.code==1){
                            //this.getWorkers();
                            this.getItem();
                        }
                        this.$emit('loaderHide');
                    });
                }
            }
        }
    }
</script>
