<template>
	<div class="chatContainer">
		<div class="logoutButton" @click="Logout()">
			Logout
		</div>
		<channels :channels="channels" @selectChannel="onSelectChannel" @createChannel="onCreateChannel"></channels>
		<div class="right">
			<chatHistory :messages="messages"></chatHistory>
			<chat @sendMessage="sendMessage"></chat>
		</div>
		<!-- dont have time for this right now so removing -->
		<!--<div class="connectedUsers">
        	<users :users="users"></users>
		</div>-->
		<newChannel :class="{hidden: !newChannelDialog}" @cancelDialog="onCancelDialog" @confirmDialog="onConfirmDialog"></newChannel>
	</div>
</template>

<script>
	import Channels from '../components/channels.vue';
	import Users from '../components/users.vue';
	import ChatHistory from '../components/chatHistory.vue';
	import Chat from '../components/chat.vue';
	import NewChannel from '../components/newChannel.vue';

	import { mapGetters, mapMutations, mapActions } from 'vuex';

	import client from '../libs/client'
	import auth from '../libs/auth';
	import service from '../libs/services';

	export default {
		data(){
			return{
				newChannelDialog: false
			}
		},

		created(){
			this.getChannels();

			// Setup events
			client.service("messages").on("created", message => {
				this.addMessage(message);
			});

			client.service("channels").on("patched", channel => {
				//updateChannel does no server calls only updates the array ref
				//this.updateChannel(channel);
				//	this.getChannels();
			});

			client.service("channels").on("created", channel => {
				this.getChannels();
			});
		},

		methods: {
			...mapMutations([
				'addMessage'
			]),

			...mapActions([
				'getChannels',
				'selectChannel',
				'updateChannel',
				'createChannel'
			]),

			onSelectChannel(channel) {
				this.selectChannel({ channel, userId: this.user._id });
			},

			onCreateChannel(){
				this.newChannelDialog = true;
			},

			onCancelDialog(){
				this.newChannelDialog = false;
			},

			onConfirmDialog(channelName){
				this.createChannel({ name: channelName, userId: this.user._id }).catch(err => {
					console.error(err);
				});
			},

			sendMessage(message) {
				// Only send if it's connected to a channel
				if (this.currentChannel._id) {
					service.sendMessage(this.currentChannel._id, this.user._id, message);
				}
			},

			Logout(){
				auth.logout();
				this.$router.push("/login");
			}
		},

		computed: {
			users() {
				const usersExist = this.currentChannel && this.currentChannel.users;
				return usersExist ? this.currentChannel.users : [];
			},
			...mapGetters([
				'channels',
				'messages',
				'currentChannel',
				'user'
			])
		},

		components: {
			Channels,
			Users,
			ChatHistory,
			Chat,
			NewChannel
		}
	}
</script>

<style>
	.hidden{
		display: none;
	}

	.chatContainer{
		display: flex;
  		flex-wrap: wrap;
		padding: 0px;
		margin: 0px;
		width: 100%;
		height: 100%;
	}

	.right{
		height: 100%;
    	width: calc(100% - 120px);
	}

	.connectedUsers{
		box-sizing: border-box;
		width: 10%;
		height: 100%;
		padding: 10px;
	}

	.logoutButton{
		position: fixed;
		right: 30px;
		top: 30px;
		border: 1px solid black;
		border-radius: 100px;
		width:90px !important;
		height: 90px !important;
		cursor: pointer;
		display: flex;
		justify-content: center;
		align-items: center;
	}
</style>
