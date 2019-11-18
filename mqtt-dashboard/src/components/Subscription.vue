<template>
<div class="subscription_container">
<div class="topic_header">
  <div class="topic" v-on:click="setTopic();toggleShowMessages();">{{subscription.topic}} - ({{subscription.messages.length}})
  </div>
  <div class="toggle_messages" :class="'show_messages_'+showMessages">
    <template v-if="!show"> ➕ </template>
    <template v-if="show"> ➖ </template>
  </div>
</div>
  <ul class="messages" v-if="showMessages">
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
      return this.show && this.subscription.messages.length 
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
}
.topic{
  padding: 10px;
  font-weight:bold;
  flex:1;
  cursor:pointer;
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
  width:30px;
  text-align:right;
  margin-right:10px;
  cursor:pointer;
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
</style>