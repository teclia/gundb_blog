<template>
    <div class="m-10">

        <div class="text-lg">
            <h1>GUNDB / VUE blog example</h1>
        </div>

        <div class="border p-4 rounded">

            <div class="pb-4">
                <h1 class="bold text-xl">Create and login user</h1>
                {{ error }}
            </div>               

            <div>
                <div class="flex flex-col">
                    <label for="username">Username</label>
                    <input type="text" id="username" v-model="username" class="p-2 border rounded">
                </div>

                <div class="flex flex-col">
                    <label for="password">Password</label>
                    <input type="text" id="password" v-model="password" class="p-2 border rounded">
                </div>            

                <div class="flex gap-2 pt-4">
                    <button @click="createUser" class=" p-2 bg-cyan-500 text-white rounded">Create User</button>
                    <button @click="loginUser" type="submit" class=" p-2 bg-cyan-500 text-white rounded">Login User</button>
                </div>
            </div>
        </div>  

        <div class="pt-5">
            <div>
                <div class="border p-4">
                    <div class="pb-4">
                    <h1 class="bold text-xl">Create a post </h1>
                </div>     

                <div class="flex flex-col">
                    <label for="title">Title</label>
                    <input type="text" id="title" v-model="messageTitle" class="p-2 border rounded">
                </div>
                <div class="flex flex-col">
                    <label for="text">Text</label>
                    <input type="text" id="text" v-model="messageText" class="p-2 border rounded">
                </div>            
                <div class="flex gap-2 pt-4">
                    <button @click="post" class=" p-2 bg-cyan-500 text-white rounded">Post message</button>
                </div>
            </div>
        </div>

            <div id="post-history" class="border mt-4 p-4">
                <div class="pb-4">
                    <h1 class="bold text-xl">Post list</h1>
                    I need only those who post the record to be able to remove
                </div>                
                <hr>
                <div v-for="post in posts" :key="post.id" class="pt-4">
                    <div class="flex justify-between p-1 rounded-md hover:bg-gray-50">
                        <div>
                            
                            <div class="text-md">{{ post.title }}</div>
                            <div class="text-lg">{{ post.text }}</div>
                            <div class="text-xs text-gray-500">{{ post.author['#']?.substring(1) }}</div>
                            
                        </div>
                        <div>
                            <button class="bg-rose-500 text-xs p-2 text-white rounded-sm" @click="removePost(post.id)">Remove</button>
                        </div>
                    </div>

                </div>
            </div>
        </div>
    </div>
  </template>
  
  <script>
    
  import GUN from 'gun/gun'
  import 'gun/sea';
  import 'gun/axe';

  
  const gun = GUN({
    peers: [
        "http://localhost:3030/gun",    
  ],
    localStorage: false,

  });

  const user = gun.get('users');
  
  export default {
    data() {
      return {
        error: '',
        messageTitle: '',
        messageText: '',
        username: '',
        password: '',
        posts: [],
      }
    },
    methods: {

        async createUser(){
            // create user with username and password
            this.user = await gun.user().create(this.username, this.password, async (ack) => {
                if (ack.err) {
                    console.log(ack.err)
                    this.error = ack.err;
                    return;
                } else {
                    console.log(ack);
                    this.error = '';
                }
            })     
        },

        async loginUser(){
            // login with username and password
            const user = await gun.user().auth(this.username, this.password, async (ack) =>  {
                if (ack.err) {
                    console.error('Authentication failed')
                    this.error = ack.err;
                    return;
                } else {
                    console.log(ack.put);
                    this.error = '';
                }
            })     
        },

        async post(){

            var post = {
                title: this.messageTitle,
                text: this.messageText,
                timestamp: Date.now()
            }

            var author = gun.get('~@'+this.username) // There should be the same `username` in Step 2
            
            gun.user().get('posts')
            .set(post) // At this step, we saved the post in a user schedule, which by default is only writable by the user
            .once(function() {
                this.get('author').put(author) // In this step, we link our post with the author (with our user)
                gun.get('posts').set(this) // At this step, we save the post with the author installed in the main graph
            })
        },

        async removePost(id){
            gun.get('posts').get(id).put(null)
        }
      
    },
    mounted() {
     
        const db = gun.get('posts')
    
        // timestamp mais novo primeiro
        db.map().on((post, id) => {

            // Remove duplicado ou null
            this.posts = this.posts.filter(p => p.id !== id);

            // Caso o post tenha sido removido
            if(!post) { return;}
            post.id = id; // Set id
            this.posts.push(post);
            this.posts.sort((a, b) => b.timestamp - a.timestamp);

        });

    }
  }
  </script>