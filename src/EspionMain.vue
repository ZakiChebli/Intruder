<template>
  
  <div id="app">
    <span v-show="!Connected" id="ConnectionBar">
    <input class="border"  
v-model="ServerAddress" placeholder="Server-address:Port" style="width:300px;">
<button class="waves-effect waves-light btn center" @click="TryToConnect()">GO</button>
Hit GO. to connect. do not refresh page when this bar goes off.
</span>
    
    <Header :Votes_Resutls="Votes_Results" />
    <EspionLobby v-show="!gamestarted"  :ws="websocket"   :PlayersReady="PlayersReady" :PlayersNames="PlayersNames"/>
    <EspionInGame :Timer="Timer":History="History" :Selected="Selected" :PlayerName="PlayerName" :Turn="Turn" v-show="gamestarted" :ws="websocket" :Faction="Faction"  :Names="PlayersNames" :Votes="Votes" :Votes_Results="Votes_Results"  :NUMBER_OF_PLAYER_IN_TEAM="NUMBER_OF_PLAYER_IN_TEAM"/>
    <Footer/>

    
  </div>
</template>

<script>
var dynamics=require("dynamics.js")
 M.AutoInit();  //Materialaze CSS init
import EspionLobby from '@/components/EspionLobby'
import EspionInGame from '@/components/EspionInGame'
import Header from '@/components/Header'
import Footer from '@/components/Footer'
//<img src="./assets/logo.png">
export default {
  name: 'EspionMain',
    data () {
    return {
      Connected:false,
      ServerAddress:"",
      websocket:"",
      gamestarted:false,

      PlayerName:'', // current player
      //Lobby
      PlayersReady: {}, // players if ready
      PlayersNames:[], // names of the players
      //Votes
      Votes:{},
      Votes_Results:{},
      Timer:-1,
      //GoToMission
      NUMBER_OF_PLAYER_IN_TEAM:88,
      Turn:"", // wich person who select team
      Faction:'',  //Faction of the player Revolu1/Spy0
      Selected:[],
      //Choice
      Decide:'', // Fail0/Success1
      dynamics:require("dynamics.js"),
      //History
      Round:1,
      History:[],

    }
  },
  methods:{
    TryToConnect(){
      let a="ws://"+this.ServerAddress;
      console.log("Trying to connect to ",a);
      this.websocket=new WebSocket(a);
      console.log(this.websocket)
      this.WaitForData()
    },
    UpdateHistory(v){
      v['Turn'] = this.Turn;
      v['Round'] = this.Round;
      v.Selected = this.Selected;
      this.History.push(v);
      this.Round++;
    },
    WaitForData(){
      var ref = this;// keep the Vue instance scope inside the callback.
        console.log(ref)
        this.websocket.onmessage = function (event) { // on receiving message from server
                let data = JSON.parse(event.data);                
                console.log('UI RECEIVE:',data)
                switch (data.type) {
                    case 'ready':
                        ref.PlayersReady=data.ready;
                        console.log('Ready type',ref.PlayersReady)
                        break;
                    case 'names':
                        ref.PlayersNames=data.names;
                        console.log('Users type',ref.PlayersNames)
                        break;
                    case 'faction':
                        ref.Faction=data.faction;
                        setTimeout(()=>{
                            ref.gamestarted=true;
                        },1000);
                        
                        console.log("Faction type",ref.Faction)
                        break;
                    case 'votes':
                        ref.Votes=data.votes;
                        console.log('Votes type',ref.Votes)
                        break;
                    case 'vote_accepted':
                        ref.Timer=data.vote_accepted.timer;
                        console.log("Vote Accepted. Time to choose Decide the faith of the mission!");
                        setTimeout(()=>{
                              ref.Timer=-1;
                        },ref.Timer*1000);
                    break;
                    case 'vote_result':
                        ref.Votes_Results=data.vote_result;
                        ref.UpdateHistory(data.vote_result);
                        console.log('Votes type',ref.Votes_Results)
                        ref.Votes={};
                        break;
                    case 'turn':
                        ref.Turn=data.turn;
                        console.log('Turn type',ref.turn)
                    break;
                    case 'selected':
                        ref.Selected=data.selected;
                        console.log('Selected type',ref.Selected)
                    break;
                    case 'params':
                        ref.NUMBER_OF_PLAYER_IN_TEAM = data.params.NUMBER_OF_PLAYER_IN_TEAM;
                        console.log('params type',ref.NUMBER_OF_PLAYER_IN_TEAM)
                    break;
                    case 'connected':
                         console.log("Server Connected");
                         ref.ServerAddress="Connected! "
                         setTimeout(()=>{
                            ref.Connected=true; 
                        },1000);
                           
                         break;
                    default:
                        console.error("unsupported event", data);
                  }//Switch
                }// Callback
        }//WaitForData      
  
  
    },//Method
  components:{Header,Footer,EspionLobby,EspionInGame}  
}
</script>

<style>

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: left;
  color: #2c3e50;
  margin-top:0px;
}


</style>
