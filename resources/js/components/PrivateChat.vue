<template>
    <v-layout row>
        <v-flex class="onlineusers" xs3>
            <v-list>
                <v-list-tile
                  v-for="friend in friends"
                  :color="(friend.id == activeFriend)?'green':''"
                  :key="friend.id"
                  @click="activeFriend = friend.id">

                    <v-list-tile-action>
                        <v-icon :color="(onlinefriends.find(onlinefriend=> onlinefriend.id === friend.id))?'green':'red'">
                            account_circle
                        </v-icon>
                    </v-list-tile-action>

                    <v-list-tile-content>
                        <v-list-tile-title>{{ friend.name }}</v-list-tile-title>
                    </v-list-tile-content>
                </v-list-tile>
            </v-list>
        </v-flex>
        <v-flex xs9 class="messages mb-5" id="privatemessage">
            <v-list>
                <!-- <v-subheader>Chat</v-subheader> -->
                <v-divider></v-divider>

                <v-list-tile class="p-3" v-for="(item, index) in allMessages" :key="index">
                    <v-layout :align-end="(user.id !== item.user.id)" column>
                        <v-flex>
                            <v-layout column>
                                <v-flex>
                                    <span class="large font-italic purple--text">{{ item.user.name }}</span>

                                    <span class="small font-italic">{{item.created_at}}</span>
                                </v-flex>

                                <v-flex>
                                    <v-chip :color="(user.id !== item.user.id)?'red':'green'" text-color="white">
                                            <v-list-tile-content>{{ item.message }}</v-list-tile-content>
                                    </v-chip>
                                </v-flex>

                               <!--  <v-flex class="caption font-italic">
                                    {{item.created_at}}
                                </v-flex> -->
                            </v-layout>
                        </v-flex>
                    </v-layout>
                </v-list-tile>
                <v-card-text class="blue--text" v-if="typingUser.name">{{ typingUser.name }} is typing</v-card-text>
            </v-list>
            <br>
            <v-footer height="auto" color="grey" fixed>
                <v-layout row>
                    <v-flex xs6 offset-xs3 justify-center align-center>
                        <v-text-field
                            rows=2
                            v-model="message"
                            label="Enter message"
                            single-line
                            @keydown="onTyping"
                            @keyup.enter="sendMessage" >

                        </v-text-field>
                    </v-flex>
                    <v-flex xs2>
                        <v-btn dark small color="green" class="mt-3 ml-2 white--text" @click="sendMessage">Send</v-btn>
                    </v-flex>
                </v-layout>
            </v-footer>
        </v-flex>
    </v-layout>
</template>

<script>
export default {
    created(){
       //this.fetchMessages()
       this.fetchUsers()

       Echo.join('plchat')
       .here((users) => {
           console.log('online users', users)
           this.onlinefriends = users
       })
       .joining((user)=> {
           console.log('joining', user.name)
           this.onlinefriends.push(user)
       })
       .leaving((user)=> {
           console.log('leaving', user.name)
           this.onlinefriends.splice(this.onlinefriends.indexOf(user), 1)
       })

       Echo.private('privatechat.'+this.user.id)
        .listen('PrivateMessageSent', (e) => {
            this.activeFriend = e.message.user_id
            this.allMessages.push(e.message)
            setTimeout(this.scrollToEnd(), 100)
        })
       .listenForWhisper('typing', (e) => {
           if(e.user.id == this.activeFriend){
                this.typingUser = e.user

                if(this.typingClock) clearTimeout()

                this.typingClock = setTimeout(()=> {
                    this.typingUser = {}
                }, 10000)
           }

       })
    },
    props: ['user'],
    data(){
        return {
            message: null,
            allMessages: [],
            users: [],
            activeFriend: null,
            typingUser: {},
            typingClock: null,
            onlinefriends: []
        }
    },
    watch: {
       activeFriend(val){
           this.fetchMessages()
       }
    },
    computed: {
        friends(){
            return this.users.filter((user) => {
               return  user.id != this.user.id
            })
        }
    },
    methods: {
        onTyping(){
            Echo.private('privatechat.'+this.activeFriend).whisper('typing', {
                user: this.user
            })
        },
        sendMessage (){
            if(!this.message) {
                return alert('Please enter message')
            }

            if(!this.activeFriend) {
                return alert('Please select friend')
            }

            //this.allMessages.push(this.message)

            axios.post('/private-messages/'+ this.activeFriend, {message: this.message})
            .then(response => {
                console.log(response)
                this.message = ''
                //this.allMessages.push(this.message)
                this.fetchMessages()
                setTimeout(this.scrollToEnd(), 100)
            })
        },
        fetchMessages() {
              if(!this.activeFriend) {
                return alert('Please select friend')
            }

            axios.get('/private-messages/'+ this.activeFriend)
            .then(response => {
                //console.log(response)
                this.allMessages = response.data
            })
        },
        fetchUsers() {
            axios.get('/users')
            .then(response => {
                this.users = response.data
                this.activeFriend = this.friends[0].id
            })
        },
        scrollToEnd(){
            document.getElementById('privatemessage').scrollTo(0, 9999)
        }
    }
}
</script>

<style scoped>
    .onlineusers, .messages {
        overflow-y: scroll;
        height: 100vh;
    }
</style>


