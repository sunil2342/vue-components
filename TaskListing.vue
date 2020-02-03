<template>
   <div class="profile_data_main">
      <div class="row manage_worker mb-3">
        <div class="col-md-6 m-auto"><h5 class="page-header-title m-auto">All Tasks</h5></div>
        <div class="col-md-6 m-auto">
          <div v-if="auth.role_id==0 && options.length>0" class="pull-right col-md-12 text-right">
              <span class="col-md-2">Agency Filter</span>
              
              <select2 name="assign" cls="form-control col-md-8 text-left" v-model="filterEmail" v-on:input="filterTasks" :placeholder="placeholder">
                <option value="default">All Agency</option>
                <option :value="agency.id" v-for="agency in options">{{agency.text}}</option>
              </select2> 
          </div>
        </div>
        <br>
      </div>
      <div class="listing-div">
        <ul class="display-inline task-filter">
          <li class="pr-2">
            <a v-on:click.stop.prevent="getTaskStatus('drafts')" class="btn btn-new-default" :class="getClass('drafts')">
              Draft ({{ taskStatus.draft?taskStatus.draft:0 }})
            </a>
          </li>
          <li class="pr-2">
             <a v-on:click.stop.prevent="getTaskStatus('estimation')" class="btn btn-new-default" :class="getClass('estimation')">
                Estimation ({{ taskStatus.estimation?taskStatus.estimation:0 }})
              </a
          </li>
          <li class="pr-2">
            <a v-on:click.stop.prevent="getTaskStatus('ready')" class="btn btn-new-default" :class="getClass('ready')">
              Ready ({{ taskStatus.ready?taskStatus.ready:0 }})
            </a>
          </li>
          <li class="pr-2">
            <a v-on:click.stop.prevent="getTaskStatus('in_progress')" class="btn btn-new-default" :class="getClass('in-progress')">
              In Progress ({{ taskStatus.in_progress?taskStatus.in_progress:0 }})
            </a>
          </li>
          <li class="pr-2">
            <a v-on:click.stop.prevent="getTaskStatus('review')" class="btn btn-new-default" :class="getClass('review')">
                Review ({{ taskStatus.review?taskStatus.review:0 }})
              </a>
          </li>
          <li class="pr-2">
              <a v-on:click.stop.prevent="getTaskStatus('reopen')" class="btn btn-new-default" :class="getClass('re-open')">
                Re-open ({{ taskStatus.reopen?taskStatus.reopen:0 }})
              </a>
          </li>
          <li class="pr-2">
              <a v-on:click.stop.prevent="getTaskStatus('done')" class="btn btn-new-default" :class="getClass('done')">
                Done ({{ taskStatus.done?taskStatus.done:0 }})
              </a>
          </li>
          <li class="pr-2 lineHeight30">
            <a v-on:click.stop.prevent="getTaskStatus('all')" class="btn">
              <img :src="config.baseUrl+img.checkbox" alt="Checked"> Check All
            </a>
          </li>
          <li class="pr-2 lineHeight30 modified-select2" v-if="priorities">
            <select2 cls="col-md-12" v-model="priority" v-on:input="priorityTasks">
              <option value="" selected="">All Prorities</option>
              <option value="critical" :data-img="config.baseUrl+'images/priority/critical.png'">Critical</option>
              <option value="high" :data-img="config.baseUrl+'images/priority/high.png'">High</option>
              <option value="medium" :data-img="config.baseUrl+'images/priority/medium.png'">Medium</option>
              <option value="low" :data-img="config.baseUrl+'images/priority/low.png'">Low</option>
              <option value="optional" :data-img="config.baseUrl+'images/priority/lowest.png'">Optional</option>
            </select2>
          </li>
        </ul>
      </div>
      <div class="row">
        <div class="col-md-12">
          <div class="bgc-white bd">
             <div class="table-responsive">
            <table class="table tasks-list">
              <thead>
                <tr>
                  <th scope="col" width="2%"></th>
                  <th scope="col" width="5%">#</th>
                  <th scope="col" width="35%">Name</th>
                  <th scope="col" width="12%">Amount</th>
                  <th scope="col" width="10%">Status</th>
                  <th scope="col" width="13%">Modified</th>
                  <th width="22%" v-show="isClient()">Action</th>
                  <th width="5%" v-show="isClient()"></th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(task, index) in tasks" class="manage_user_list" v-on:click="taskDetails($event)" v-bind:id="task.id">
                    <td><img :src="taskPriorityIcon(index)" width="20px;" :title="taskPriorityName(index)"></td>
                    <td class="serial-number" scope="row">{{ task.id }}</td>
                    <td>{{ task.name }}</td>
                    <td>
                        {{ task['task_detail'] ? currency+(task['task_detail']['estimation']*task['rate']).toFixed(2):(task.price? currency+task.price.toFixed(2):'')  }}</td>
                    <td class="delete">
                        <div class="status-wrapper status-drafts" v-if="task.status==0">Draft</div>
                        <div class="status-wrapper status-estimation" v-else-if="task.status==3">Estimation</div>
                        <div class="status-wrapper status-ready" v-else-if="task.status==4">Ready</div>
                        <div class="status-wrapper status-in-progress" v-else-if="task.status==5">In-Progress</div>
                        <div class="status-wrapper status-review" v-else-if="task.status==6">Review</div>
                        <div class="status-wrapper status-re-open" v-else-if="task.status==7">Re-Open</div>
                        <div class="status-wrapper status-done" v-else-if="task.status==8">Complete</div>
                        <div class="status-wrapper status-drafts" v-else >Draft</div>
                    </td>
                    <td>
                      {{ task.created_at | formatDate }}
                    </td>
                    <td class="website-link" v-show="isClient()">
                      <ul class="action-list">
                        <li v-show="task.status < 3">
                          <a class="status-estimation" v-on:click.stop.prevent="askForEstimation(index)">
                            <span class="wps_paypal_icon">
                              <img :src="config.baseUrl+'images/calc.png'">
                            </span> Ask estimation
                          </a>
                        </li>
                        <li v-show="task.status==6 || task.status==7">
                          <a href="javascript:void(0);" class="status-review">
                            <span class="wps_paypal_icon">
                              <img :src="config.baseUrl+'images/search.png'">
                            </span> Review the Task
                          </a>
                        </li>
                      </ul>
                    </td>
                    <td  class="website-link" v-show="isClient()">
                      <ul class="action-list">
                        <li>
                          <a href="javascript:void(0);" v-on:click.stop.prevent="deleteTask(index)">
                            <span>
                              <img :src="config.baseUrl+'images/trash.png'">
                            </span>
                          </a>
                        </li>
                      </ul>
                    </td>
                </tr>
                <tr v-if="tasks.length == 0">
                  <td :colspan="isClient()?8:6" class="text-center">No record found</td>
                </tr>
              </tbody>
              <tfoot>
                <tr>
                  <td :colspan="isClient()?8:6" class="text-right">      
                    <div class="text-center">
                      <div class="paginate-count pagination pull-left">Total Records : {{ tasks.length }}</div>
                    </div>
                  </td>
                </tr>
              </tfoot>
            </table>
          </div>
          </div>
        </div>
      </div>
      <div v-if="isClient()">
          <model-popup v-if="showEstimationModal" :class="showEstimationModal?'show':''" @close="showEstimationModal = false">
              <h3 slot="header">Move Task Under Estimation</h3>
              <div slot="body">
                  <form v-on:submit.prevent="askForEstimation(itemIndex)">
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
</template>
<script>
    export default {
        props: ['status'],
        data(){
            return{
                tasks: [],
                pagination: {
                    'current_page': 1
                },
                taskStatus: {},
                img:{'checkbox':'images/check.png'},
                defaultStatus: '',
                cPage:1,
                currency:'',
                placeholder:"Select Agency",
                options:[],
                showEstimationModal:false,
                teamOptions:[],
                defaultTeam:'web',
                itemIndex:'',
                priorities:{},
                priority:(this.$route.query.priority?this.$route.query.priority:''),
                filterEmail:(this.$route.query.filter?this.$route.query.filter:'default'),
                auth : this.$auth.user()
            }
        },
        computed: {     

        },
        created: function()
        {
          document.title = this.$route.meta.title;
          //console.log(this.auth);
          this.fetchItems();
          if(this.auth.role_id==0){
            this.getAgency();
          }
        },
        methods: {
          isClient: function () {
            return this.auth.role_id==1?true:false;
          },
          taskPriorityIcon:function(index){
            let prior = this.priorities.length-this.tasks[index].priority-1;
            return this.priorities[prior].img;
          },
          taskPriorityName:function(index){
            let prior = this.priorities.length-this.tasks[index].priority-1;
            return this.priorities[prior].text;
          },
          getClass(defCl){
            let cl = '-active';
            let clStatus = '';

            switch (defCl) {
              case 'in-progress':
                clStatus = 'in_progress';
                break;
              case 're-open':
                clStatus = 'reopen';
                break;
              default:
                clStatus = defCl;
            }
            if(this.$route.params.status){
              cl = '';
              if(this.$route.params.status==clStatus){
                cl = '-active';
              }
            }
            return 'btn-filter-'+defCl+cl;
          },
          getAgency(){            
            let uri = '/agency';
            this.axios.get(uri).then((response) => {
              //console.log(response.data.agencyList);
              this.options = response.data.agencyList;
            });
          },
          fetchItems()
          {
              let st = (this.status?this.status:"all");
              this.getTasks(st);
          },
          fetchTasks(type){
            this.cPage = this.pagination.current_page;
            this.$emit('loaderShow');
            let email = this.filterEmail;
            if(type!='all'){
              this.img.checkbox = 'images/uncheck.png';
              this.defaultStatus = type;
              //this.$router.replace("/task/"+type);
            }else if(this.defaultStatus!='all'){
              this.img.checkbox = 'images/check.png';
              this.defaultStatus = type;
              //this.$router.replace("/tasks");
            }
            let uri = '';
            uri = '/tasks/'+email+'/'+type;   
            if(email=='web' || email=='default'){
              uri = '/tasks/'+this.auth.email+'/'+type; 
            }
            if(email!='default'){
              uri = uri+'?filter='+email;
            }
            if(email!='default' && this.priority!=''){
              uri = uri+'&priority='+this.priority;
            }else if(this.priority!=''){
              uri = uri+'?priority='+this.priority;
            }
            
            this.getClass(type);
            this.axios.get(uri).then((response) => {
                this.$emit('loaderHide');
                this.tasks = response.data.data;
                this.pagination = response.data.pagination;
                this.taskStatus = response.data.aTaskStatus;
                this.currency = response.data.currency.symbol;
                this.priorities = response.data.priorities;
                if(this.auth.parent_id && response.data.team){
                    this.teamOptions = response.data.team.withAgency;
                }else if(response.data.team){
                    this.teamOptions = response.data.team.withOutAgency;
                }
            });
          },
          getTasks(type){
            if(this.defaultStatus!=type || (this.pagination.current_page!=0 && this.pagination.current_page!=this.cPage)){
              this.fetchTasks(type);
            }
          },
          getTaskStatus(type){

            let queryString = {};
            if(this.priority!=''){
              queryString['priority'] = this.priority;
            }
            if(this.filterEmail!='default'){
              queryString['filter'] = this.filterEmail;
            }
            this.$router.replace({name: (type=='all'?'tasks':'task-status'), params: (type=='all'?'':{ status: type }),query:queryString});
          },
          deleteTask(index)
          {
              let uri = '/task/delete/'+this.tasks[index].id;
              this.$emit('loaderShow');
              this.tasks.splice(index, 1);
              this.axios.delete(uri).then((response) => {this.$emit('loaderHide');});
              this.defaultStatus = "";
              this.fetchItems();              
          },
          taskDetails(e){
              this.$emit('loaderShow');
              this.$router.replace({ name: "task-details",params: { id: $(e.target).parents('tr.manage_user_list').attr('id') }});
          },
          callEstimationPopup(index){
            this.itemIndex = index;
            this.showEstimationModal = true;
          },
          askForEstimation(index){
            let uri = '/task/ask_estimation/'+this.auth.email+'/'+this.tasks[index].id;
            this.$emit('loaderShow');
            this.axios.post(uri,{team:this.defaultTeam}).then((response) => {
              this.$emit('loaderHide');
              if(response.data.code==1){
                this.defaultStatus = "";
                this.fetchItems();
              }
            });
          },
          filterTasks(email){
            this.$emit('loaderShow');
            let queryString = {};
            if(this.priority!=''){
              queryString['priority'] = this.priority;
            }
            if(email!='default'){
              queryString['filter'] = email;
            }
            let st = (!this.status?'all':this.status);
            this.$router.push({name: (st=='all'?'tasks':'task-status'), params: (st=='all'?'':{ status: this.status }),query:queryString});
            this.fetchTasks((this.status?this.status:"all"));
          },
          priorityTasks(val){
            this.$emit('loaderShow');
            let st = (this.status?this.status:"all");
            let queryString = {};
            if(this.priority!=''){
              queryString['priority'] = this.priority;
            }
            if(this.filterEmail!='default'){
              queryString['filter'] = this.filterEmail;
            }
            this.$router.replace({name: (st=='all'?'tasks':'task-status'), params: (st=='all'?'':{ status: st }),query:queryString});
            this.fetchTasks(st);
          }
        }
    }
</script>
