<template>
  <div>
      <v-container class="mt-5">
                    <v-layout row wrap v-for="data in datas" :key='data.id'>
               <v-flex xs12 md6 class="text-center">
                  <v-responsive>
                      <img :src="`/anime/${data.photo}`" alt="" style="max-height:300px;max-width:200px;">
                  </v-responsive>
               </v-flex>
               <v-flex xs12 md6 class="left mt-5">
                  <h5 class="subheading">{{data.name}}</h5>
                  <h5 class="subheading">{{data.tag}}</h5>
                  <h5 class="subheading">{{data.date}}</h5>
               </v-flex>
               <v-flex xs12 md12 class="text-left mt-5">
                   <h5 class="headline">PLOT</h5>
                   <h6 class="subheading">{{data.story}}</h6>
               </v-flex>
          </v-layout><br><hr>
      </v-container>
      <v-container>
          <v-form ref="form">
              <v-layout row wrap >
              <v-flex xs6 offset-xs3 md3 class="ml-5">
                 <v-text-field outlined v-model="episode" label="Episode" :rules="epRules" required autofocus>
                 </v-text-field>       
              </v-flex>
              <v-flex xs6 offset-xs3 md3 class="ml-5">
                     <v-text-field outlined v-model="name" label="Ep Name" :rules="nameRules" required >
                 </v-text-field> 
              </v-flex>
              <v-flex xs6 offset-xs3 md3 class="ml-5">
                     <v-text-field outlined v-model="link" label="Link"  @keyup.enter="addepisode" >
                 </v-text-field> 
              </v-flex>
              
                    <v-btn class="info white--text ml-6 mt-4" @click="isedit ? editep():addepisode()">  
                        <span>{{isedit ? 'Update':'Add'}}</span>
                        <v-icon>{{isedit ? 'update':'add'}}</v-icon>
                    </v-btn>
                    <v-btn v-if="isedit==true" class="info white--text ml-6 mt-4" @click="clearedit">
                        <span>Go Add</span>
                        <v-icon>add</v-icon>
                    </v-btn> 
                     <v-btn style="text-decoration:none;" class="red white--text ml-6 mt-4" :to="`/animeframe/myprojects/viewseason/${mid}`">Back</v-btn>
              
          </v-layout>
          </v-form>
          <v-layout row wrap class="mt-4">
               <v-flex xs12 md6  v-for="ep in episodes" :key="ep.id" class="mb-2">
                   <v-dialog :v-model="`dialog`+ep.id" width="600px" height="auto">
                       <template v-slot:activator={on}>
                              <v-btn text class="primary--text"  v-on="on">
                               <span>{{ep.episode}} . {{ep.ep_name}}</span>
                              </v-btn>
                       </template>
                      <v-row>
                          <v-col cols="12" md="12">
                                  <v-card>
                                    <v-card-text class="text-center">
                                        <v-btn class="primary" :to="`/aframe/admin/watch/${ep.id}`">Go</v-btn>
                                        <v-btn class="info ml-5" @click="showeditep(ep)">Edit</v-btn>
                                        <v-btn class="red white--text ml-5" @click="delep(ep.id)">Delete</v-btn>
                                    </v-card-text> 
                                 </v-card>
                          </v-col>
                      </v-row> 
                   </v-dialog>
               </v-flex>
          </v-layout>
      </v-container>
  </div>
</template>

<script>
export default {
data()
{
    return{
    
    datas:[],
    mid:this.$route.params.mid,
    sid:this.$route.params.sid,
    seasons:[],
    name:'',
    episode:'Ep ',
    link:'',
    episodes:[],
    isedit:false,
    epid:'',
    epRules:[
        v=>v.length >3 || 'Episode field is required!'
    ],
    nameRules:[
        v=>!!v || 'Name field is required!'
    ],
    }
},
methods:{
     async getdata()
    {
      await axios.get('/animeframe/getdata/'+this.mid)
      .then((resp)=>{
          this.datas=resp.data;
          
      })
    },
     async geteachseason()
    {
        await axios.get('/animeframe/geteachseason/'+this.sid)
        .then((resp)=>{
            this.seasons=resp.data[0];
        })
    },
    async addepisode()
    {
        if(this.$refs.form.validate())
        {
            var formdata=new FormData();
        formdata.append('episode',this.episode)
        formdata.append('link',this.link)
        formdata.append('ep_name',this.name)
        formdata.append('anime_id',this.mid)
        formdata.append('season_id',this.sid)
        await axios.post('/animeframe/addepisode',formdata)
        .then((resp)=>{
            this.$refs.form.reset();
            this.episode="Ep ";
            this.link="";
            this.name="";
            this.getepisode()
            const Toast=Swal.mixin({
                toast:true,
                position:'top-right',
                iconColor:'white',
                customClass:{
                    popup:'colored-toast'
                },
                showConfirmButton:false,
                timer:1500,
                timerProgressBar:true
            })
            Toast.fire({
                icon:'success',
                title:'Success'
            })
        })
        }
    },
    async getepisode()
    {
        await axios.get(`/animeframe/getepisode/${this.sid}`)
        .then((resp)=>{
         this.episodes=resp.data;
        })
    },
    clearedit()
    {
            this.episode="";
            this.link="";
            this.name="";
            this.isedit=false;
    },
    async showeditep(ep)
    {
       this.episode=ep.episode;
       this.link=ep.link;
       this.name=ep.ep_name;
       this.epid=ep.id;
       this.isedit=true;
    },
     async editep()
    {
      var formdata=new FormData()
      formdata.append('episode',this.episode)
        formdata.append('link',this.link)
        formdata.append('ep_name',this.name)
        formdata.append('anime_id',this.mid)
        formdata.append('season_id',this.sid)
        formdata.append('_method','PUT');
        await axios.post(`/aframe/episode/update/${this.epid}`,formdata)
        .then((resp)=>{
            this.$refs.form.reset();
             this.episode="Ep ";
             this.link="";
             this.name="";
             this.getepisode();
            const Toast=Swal.mixin({
                toast:true,
                iconColor:'white',
                position:'top-right',
                customClass:{
                    popup:'colored-toast'
                },
                showConfirmButton:false,
                timerProgressBar:true,
                timer:1500
            })
            Toast.fire({
                icon:'success',
                title:'Edited'
            })
        })
    },
    async delep(id)
    {
        var formdata=new FormData();
        formdata.append('_method','DELETE');
        await axios.post(`/aframe/episode/delete/${id}`,formdata)
        .then((resp)=>{
            this.getepisode();
            const Toast=Swal.mixin({
              toast:true,
              position:'top-right',
              iconColor:'red',
              customClass:{
                  popup:'colored-toast'
              },
              showConfirmButton:false,
              timer:1500,
              timerProgressBar:true,
            })
            Toast.fire({
                icon:'error',
                title:'Deleted'
            })
        })
    }
},
mounted()
{
    this.getdata();
    this.geteachseason();
    this.getepisode();
}
}
</script>

<style lang="scss" scoped>
@media only screen and (max-width:600px)
{
    .left{
        text-align: center;
    }
}
</style>