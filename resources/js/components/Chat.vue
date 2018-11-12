<template>
    <v-layout row>
        <v-flex xs12 sm6 offset-sm3>
            <v-card class="mb-4" dark="">
                <v-list>
                    <v-subheader>Chat</v-subheader>
                    <v-divider></v-divider>

                    <v-list-tile class="p-3" v-for="(item, index) in allMessages" :key="index">
                        <v-layout :align-end="(user.id !== item.user.id)" column>
                            <v-flex>
                                <v-layout column>
                                    <v-flex>
                                        <span class="small font-italic">{{ item.user.name }}</span>
                                    </v-flex>

                                    <v-flex>
                                        <v-chip :color="(user.id !== item.user.id)?'red':'green'" text-color="white">
                                             <v-list-tile-content>{{ item.message }}</v-list-tile-content>
                                        </v-chip>
                                    </v-flex>

                                    <v-flex class="caption font-italic">
                                        {{item.created_at}}
                                    </v-flex>
                                </v-layout>
                            </v-flex>
                        </v-layout>
                    </v-list-tile>
                </v-list>
            </v-card>
        </v-flex>


        <v-footer height="auto" color="grey" fixed>
            <v-layout row>
                <v-flex xs6 offset-xs3 justify-center align-center>
                    <v-text-field
                      rows=2
                      v-model="message"
                      @keyup.enter="sendMessage"
                      label="Enter message"
                      single-line >

                    </v-text-field>
                </v-flex>
                <v-flex xs2>
                    <v-btn dark small color="green" class="mt-3 ml-2 white--text" @click="sendMessage">Send</v-btn>
                </v-flex>
            </v-layout>
        </v-footer>
    </v-layout>
</template>

<script>
export default {
    created(){
       this.fetchMessages()
    },
    props: ['user'],
    data(){
        return {
            message: null,
            allMessages: []
        }
    },
    methods: {
        sendMessage (){
            if(!this.message) {
                return alert('Please enter message')
            }

            //this.allMessages.push(this.message)

            axios.post('/messages', {message: this.message})
            .then(response => {
                console.log(response)
                this.fetchMessages()
                //setTimeout(this.scrollToEnd(), 100)
            })
        },
        fetchMessages() {
            axios.get('/messages')
            .then(response => {
                console.log(response)
                this.allMessages = response.data
            })
        },
        scrollToEnd(){
            window.scrollTo(0, 9999)
        }
    },
    mounted(){
        Echo.private('newchat')
        .listen('MessageSent', (e) => {
            this.allMessages.push(e.message)
            //setTimeout(this.scrollToEnd(), 100)
        })
    }
}
</script>

