<template>
<div>
<div class="connection_settings_form">
    <div class="logo"><b>mqtt://</b> dashboard</div>
    <div class="connection_form_controls">
        <label>Host: </label><input v-model="host" id="data_host">
        <label>Port: </label><input v-model="port" id="data_port">
        <label>Client Id: </label><input v-model="clientId">
        <button id="connect" v-on:click="newConnection">Connect</button>
    </div>
</div>
<template v-if="client.options">
    <div class="client_details">Connected to: {{client.options.href}}  | Client ID: {{client.options.clientId}}</div>
    <div class="topic_form">
        <div :class="'topic_'+isSubscribed"></div>
        <input v-model="topic" id="topic_form_topic" placeholder="type/topic/here...">
        <div class="field_option"><label>qos:</label><input v-model="subscribe_qos"></div>
        <template v-if="isSubscribed">
            <button v-on:click="onUnsubscribe" id="subscribe_button" v-bind:class="'subscribed_'+isSubscribed">Unsubscribe</button>
        </template>
        <template v-if="!isSubscribed">
            <button v-on:click="onSubscribe" id="subscribe_button"  v-bind:class="'subscribed_'+isSubscribed">Subscribe</button>
        </template>
        
    </div>


    <div class="publish_form">
        <div class="message_container">
            <label>Publish message: </label>
            <textarea v-model="message" class="message_input"></textarea>
        </div>
        <div class='message_controls'>
            <div><input type="checkbox" v-model="retain"><label>Retain</label></div>
            <button v-on:click="onPublish" class="publish_button">Publish</button>
        </div>
    </div>




    <div class="subscriptions">
        <h1 v-if="subscriptionCount">Subscriptions</h1>
        <div v-for="(subscription, index) in clientSubscriptions" v-bind:key="index">
            <app-subscription :subscription="subscription" v-on:setTopic="onSetTopic"></app-subscription>
        </div>
    </div>

    <div class="toggle_client" v-on:click="show_client_details=!show_client_details">Show client details >></div>
    <div v-if="show_client_details">
        <pre>
        {{client}}
        </pre>
    </div>


</template>
<div v-else class="no_connection_message">Connect to a client to publish or subscribe.</div>

</div>
</template>

<script>

import Subscription from './Subscription'
var mqtt=require('mqtt');
export default {
  name: 'HelloWorld',
  props: {
    msg: String
  },
  components: {
     'app-subscription': Subscription 
  },
  data: function () {
    return {
      msglog: [],
      host: 'mqtt://test.mosquitto.org',
      port: '8080',
      topic: 'test/topic/',
      subscribe_qos: 0,
      clientId: 'client_' +  Date.now().toString(),
      message: '',
      retain:false,
      client: {},
      subscriptions: [],
      show_client_details: false
    }
  },
  created(){
    },
    methods: {
        newConnection(){
            this.client = mqtt.connect(this.host+':'+this.port,{clientId:this.clientId})
            this.client.on('message', this.onMessage)
        },
        onMessage(topic,message){
            this.subscriptions.find((sub)=>{
                return sub.topic == topic
            }).messages.push(message.toString())
            // this.msglog.push(message.toString())
        },
        onSubscribe(){
            // Find the topic if it already exists:
            if (!this.subscriptions.find((sub)=>{
                if (sub.topic == this.topic){
                    if (sub.status){
                        alert('Already subscribed to that topic.')
                    }
                    else{
                       this.client.subscribe(this.topic) 
                       sub.status = true
                    }
                    return true
                }
                else{
                    return false
                }
            })){
                // If no sub was found:
                this.client.subscribe(this.topic, {qos:parseInt(this.subscribe_qos)})
                this.subscriptions.push({host:this.host,port:this.port,clientId:this.clientId,topic:this.topic,messages:[],id:this.topic,revision:0, status:true})
            }

            this.client.__ob__.dep.notify()

        },
        onUnsubscribe(){
            this.client.unsubscribe(this.topic)
            
            this.subscriptions.find((subscription)=>{
                if (subscription.topic == this.topic){
                  subscription.status = false
                  return true
                }
              });

            this.client.__ob__.dep.notify()
        },
        onPublish(){
            this.client.publish(this.topic,this.message,{retain:this.retain})
        },
        onSetTopic(newTopic){
            this.topic = newTopic
        }
    },
    computed: {
        isSubscribed(){
            if (this.subscriptions.find((sub)=>{return sub.topic == this.topic && sub.status})){
                return true
            }
            else {
                return false
            }
        },
        subscriptionCount(){
            return this.subscriptions.length
        },
        clientJson(){
            return JSON.stringify(this.client, null, 2);
        },
        clientSubscriptions(){
            return this.subscriptions.map((subscription)=>{
                subscription['client'] = this.client._resubscribeTopics[subscription.topic]
                return subscription
            })
            
        }
    }
}
</script>

<style>
.subscriptions{
    display:grid;
    padding:20px;
}

body{
    margin:0;
    padding:0;
}
.connection_settings_form{
    background-color:#304051;
    padding: 20px 20px;
    color:white;
    display:flex;
    align-items: center;
    flex-wrap: wrap;
}
.logo{
    flex:1;
    color:#899fba;
    font-size:1.2em;
    font-weight:bold;
    white-space: nowrap; 
}
.logo b{
    color:white;
}
.connection_form_controls{
 white-space: nowrap;   
}
.connection_settings_form label{
    font-weight:bold;
    margin-right:10px;
}
.connection_settings_form input{
    margin-right:20px;
    padding: 5px 10px;
    border:none;
    border-radius:2px;
}
#data_host{
    min-width:300px;
}
#data_port{
    width:40px;
    text-align:center;
}
#connect{
padding: 5px 20px;
border-radius:2px;
border: none;
background-color:#8aa1ba;
color:#232e3b;
cursor:pointer;
}
.no_connection_message{
    padding:40px;
    text-align:center;
    font-size:1.4em;
}
.client_details{
    padding: 10px 20px;
    background-color:#F8F8F8;
    border-bottom:solid 1px #cccccc;
    font-size:.8em;
    color:#666666;
}


.topic_form{
    display:flex;
    border:none;
    background-color:#f7f9fc;
    border-bottom:solid 1px #cccccc;
    align-items:center;
}
.field_option{
    display:flex;
    align-items:stretch;
    border-radius:4px;
    border:solid 1px #cccccc;
    text-align:center;
}
.field_option label{
    padding:10px;
    border-right: solid 1px #cccccc;
}
.field_option input{
    text-align:center;
    width:1em;
    padding: 0px 15px;
    border-radius:2px;
    border:none;
}
#topic_form_topic{
  background:none;
    padding:10px 20px;
    font-weight:bold;
    font-size:1.4em;
    flex:1;
    color:#495F7A;
    border:none;
}
.topic_true{
    background-color: #4a665b;
    width: 15px;
    height: 15px;
    margin: auto;
    border-radius: 100px;
    margin-left: 20px;
}
.topic_false{
    background-color: #FA857E;
    width: 15px;
    height: 15px;
    margin: auto;
    border-radius: 100px;
    margin-left: 20px;
}
.publish_form{
    display:flex;
    padding: 20px;
    border-bottom:solid 1px #cccccc;
    align-items:stretch;
    background-color:#E4ECF5;
}
#subscribe_button{
    box-sizing:border-box;
    margin:10px;
    padding:10px 40px;
    color:white;
    font-weight:bold;
    cursor:pointer;
    border-radius:4px;
    
}
.subscribed_false{
    background-color:#4a665b;
    border:solid 1px #4a665b;
}
.subscribed_true{
    background-color:#FA857E;
    border:solid 1px #C7524B;
}
.message_input{
    padding: 10px 20px;
    width:100%;
    box-sizing:border-box;
    border-radius:4px;
    border:solid 1px #7D90A6;
}
.message_container{
    flex:1;
    margin-right:20px;
}
.message_container label{
    font-weight:bold;
    display:block;
    margin-bottom:10px;
}
.message_controls{
    display:flex;
    flex-direction: column;
    width:200px;
}
.publish_button{
    margin:10px;
    margin-left:0px;
    padding: 15px 40px;
    border-radius:2px;
    border:solid 1px #cccccc;
    font-weight:bold;
    cursor:pointer;
    background-color: 	#F8F8F8;
      border:solid 1px  	#B8B8B8;
    -webkit-box-shadow: 0px 1px 1px 0px rgba(184,184,184,1);
    -moz-box-shadow: 0px 1px 1px 0px rgba(184,184,184,1);
    box-shadow: 0px 1px 1px 0px rgba(184,184,184,1);
}
.toggle_client{
    cursor:pointer;
    padding: 10px 20px;
    border-top:solid 1px #cccccc;
    text-decoration:underline;
}
</style>