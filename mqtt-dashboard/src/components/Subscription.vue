<template>
<div class="subscription_container">
<div class="topic_header" v-on:click="toggleShowMessages();"> 
  <div class="topic" v-on:click="setTopic();">
    <div :class="'subscription_status_'+subscription.status"></div><div>{{subscription.topic}} — ({{subscription.messages.length}}) <template v-if="subscription.client"> — qos: {{subscription.client.qos}}</template></div>
  </div>
  <div class="toggle_messages" :class="'show_messages_'+showMessages">
    <template v-if="!show"> + </template>
    <template v-if="show"> - </template>
  </div>
</div>
  <ul class="messages" v-if="showMessages">
    <li v-if="!subscription.messages.length"><i>No messages for this subscription...</i></li>
    <li v-for="(message,index) in subscription.messages" v-bind:key="index">{{message}}</li>
</ul>
</div>
</template>

<script>

export default {
  name: 'HelloWorld',
  props: ['subscription'],
    data: function () {
      return {
          buffer: [],
          show:true
      }
  },
  methods: {
    toggleShowMessages(){
      this.show = !this.show
    },
    setTopic(){
      this.$emit('setTopic',this.subscription.topic) 
    }
  },
  created(){
   },
  computed: {
    showMessages(){
      return this.show
    }
  }
}
</script>

<style>
.subscription_container{
  border:solid 1px  	#B8B8B8;
  background-color: 	#F8F8F8;
    -webkit-box-shadow: 0px 1px 1px 0px rgba(184,184,184,1);
    -moz-box-shadow: 0px 1px 1px 0px rgba(184,184,184,1);
    box-shadow: 0px 1px 1px 0px rgba(184,184,184,1);
    margin-bottom:10px;
    border-radius: 2px;
}
.topic{
  padding: 10px;
  font-weight:bold;
  flex:1;
  cursor:pointer;
  display: flex;
  align-items: center;
}
.messages{
  margin:0px;
  padding:5px;
  background-color:white;
}
.messages li{
  list-style:none;
  padding: 2px;
  min-height:1em;
}
.toggle_messages{
  text-align:center;
  margin-right:10px;
  cursor:pointer;
  font-weight:bold;
}
.show_messages_true{
  content:'goodbye';
}
.show_messages_false{
  content:'goodbye';
}
.topic_header{
  display:flex;
  align-items:center;
}
div[class^="subscription_status_"]{
  font-weight:bold;
  border-radius:100px;
  margin-right:10px;
  width:10px;
  height:10px;
} 
.subscription_status_false{
  background-color:#C7524B;
}
.subscription_status_true{
  background-color:#4a665b;
}
</style>