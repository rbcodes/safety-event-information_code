<template>
  <div class="main-container">
     <h1 style="margin-bottom: 30px;">Voilations List</h1>
     <b-row>
       <b-col v-for="(item, index) in items" :key="index" lg="3" md="4" sm="6">
         <b-card 
           :img-src="imageLoading ? 'https://picsum.photos/500' : item.image_url"  @load="imageLoading = false" 
           :img-alt="item.title" 
           @click="openModal(item)"
           img-top 
           class="mb-2">
           <b-card-text>
             <p>
               <strong>{{ "#"+item.id}}</strong>
               <strong><span v-if="item.rule && item.violations_categories && item.violations_categories[0]">{{ " | "+item.violations_categories[0].category.name }}</span></strong>
               <strong><span v-if="item.violations_tags && item.violations_tags.tag && item.violations_tags.tag.emoji[0]"></span>{{ " | "+item.violations_tags[0].tag.emoji }}</strong>
             </p>
             <div class="d-flex justify-content-between align-items-center">
               <div style="padding-top: 10px;">  
                   <p> {{ item.event_time | formatDateWithoutTime }}</p>   
               </div>
               <div class="avatars d-flex align-items-center">
                 <span class="avatar" v-for="(stakeholder, i) in item.violations_stakeholders" :key="i">
                       <img 
                         src="https://picsum.photos/100" 
                         width="50" 
                         height="50"  
                         v-b-tooltip.hover.top= "stakeholder.user.first_name"/>
                 </span>
               </div>
             </div> 
                  
           </b-card-text>
         </b-card>
       </b-col>
     </b-row>
 
   <b-modal v-model="showModal" size="lg" hide-footer>
         <div v-if="modalItem">
           <video controls autoplay width="100%">
             <source :src="videoLoading ? 'https://test-videos.co.uk/vids/bigbuckbunny/mp4/h264/720/Big_Buck_Bunny_720_10s_1MB.mp4' : item.video_url"  @load="videoLoading = false" type="video/mp4">
           </video>
           <div class="d-flex justify-content-between flex-wrap align-items-center" style="margin-top: 10px;">
             <p><strong>{{ "Event #"+modalItem.id}}</strong></p>
             <p><strong>{{ modalItem.event_time | formatDate }}</strong></p>
           </div>
           <p>
             <strong>{{ modalItem.violations_categories[0] ? modalItem.violations_categories[0].category.name : '' }}</strong> 
             <strong>  |  {{ modalItem.rule ? modalItem.rule.name : '' }}</strong>
             <strong>  |  {{ modalItem.rule ? modalItem.rule.rules_cameras[0].camera.name : '' }}</strong>
           </p>
           <p style="margin-top: 25px;" v-if="modalItem.violations_stakeholders && modalItem.violations_stakeholders[0]"><strong>Stakeholders</strong></p>
           <div v-for="(stakeholder, i) in modalItem.violations_stakeholders" :key="i">
             <div class="user-profile d-flex align-items-center" style="margin-bottom: 10px; margin-left: 20px;">
               <div>
                 <img :src="imageLoading ? 'https://picsum.photos/500' : stakeholder.user.profile_pic_url"  @load="imageLoading = false" alt="Profile Pic" class="profile-pic" width="40" height="40" style="border-radius: 50%; margin-right: 10px;">
               </div>
               <div class="align-items-center" style="padding-top: 10px;">
                 <p>{{ stakeholder.user.first_name }} {{ stakeholder.user.last_name }}</p>
               </div>
             </div>
           </div>
           
           <!-- Comment Section -->
           <p style="margin-top: 25px;"  v-if="modalItem.comment_threads_aggregate && modalItem.comment_threads_aggregate.aggregate.count > 0"><strong>Comments</strong></p>
           <div v-if="modalItem.comment_threads" style="margin-left: 20px;">
             <div v-for="(commentThread, i) in modalItem.comment_threads" :key="i" class="comment">
               <div class="user-profile d-flex align-items-center">
                 <img :src="imageLoading ? 'https://picsum.photos/500' : commentThread.user.image_url"  @load="imageLoading = false" alt="Profile Pic" class="profile-pic" width="40" height="40" style="border-radius: 50%; margin-right: 10px;">
                 <div style="background-color: #EBEBEB ; border-radius: 5px; padding: 5px; margin-bottom: 10px; margin-right: 40px; width: 100%;">
                   <p class="user-name m-0"> <strong> {{ commentThread.user.first_name }} {{ commentThread.user.last_name }}</strong></p>
                   <p class="comment-text">{{ commentThread.message }}</p>
                 </div>             
               </div>            
             </div>
           </div>
 
         
         </div>
   </b-modal>
 </div>
 
 </template>
 <style>
 .main-container {
   padding: 20px;
 }
 .avatar img {
   border-radius: 50%;
   position: relative;
   left: -5px;
   margin-left: -25px;
   z-index: 1;
   border: 2px solid #fff;
 } 
 .avatars {
   direction: rtl;  /* This is to get the stack with left on top */
   text-align: left;  /* Now need to explitly align left */
   padding-left: 25px;  /* Same value as the negative margin */
 }
 .translucent-image {
   opacity: 0.7; /* Set the desired transparency level */
   position: relative;
 }
 .custom-modal .modal-dialog {
   max-width: 80%; /* Adjust the width as needed */
 }
 
 </style>
 
 
 <script>
 import axios from 'axios';
 import { BModal } from 'bootstrap-vue';
 
 
   export default {
     data: function () {
       return {
         items: [],
         imageLoading: true,
         videoLoading: true,
         showModal: false,
         modalItem: null,
       }
     },
     components: {
       BModal,
     },
     mounted() {
     // Make an Axios request to fetch data from the JSON file
     axios.get('/safety_event_information/eventData.json')
       .then((response) => {
         console.log(response.data.data.violations_list);
         this.items = response.data.data.violations_list; // Update the items with the fetched data
       })
       .catch((error) => {
         console.error('Error loading data from JSON file:', error);
       });
      },
     methods: {
       addToCart(index) {
       // Implement your add to cart logic here
       console.log(`Added item ${this.items[index].title} to cart.`);
       },
       getImageUrl(item) {
         // You can modify the image URL based on your needs
         // For example, if images are stored in a specific directory
         return `/path/to/images/${item.image_url}`;
       },
       viewDetails(index) {
         // Perform action to view details for the specific card, based on its index
         console.log("Viewing details for item at index: ", index);
         // You can implement further actions here based on the card clicked
       },
       openModal(item) {
       this.modalItem = item;
       this.showModal = true;
       },
       closeModal() {
         this.showModal = false;
       },
     },
     watch: {
       'imageLoading': function() {
         console.log("Image Loaded")
         this.imageLoading = true;
       },
       'videoLoading': function() {
         console.log("Video Loaded")
         this.videoLoading = true;
       }
     },
     computed: {
       formattedClubs() {
           return this.clubs.reduce((c, n, i) => {
               if (i % 4 === 0) c.push([]);
               c[c.length - 1].push(n);
               return c;
           }, []);
       }
     },
     filters: {
       formatDate: function (timestamp) {
         const options = {
           year: 'numeric',
           month: 'long',
           day: 'numeric',
           hour: '2-digit',
           minute: '2-digit',
           second: '2-digit',
           hour12: true,
         };
         const formattedDate = new Date(timestamp).toLocaleString('en-US', options);
 
         return formattedDate.replace(' at', ',');
       },
       formatDateWithoutTime(date) {
         if (!date) return '';
         const options = { year: 'numeric', month: 'long', day: 'numeric' };
         return new Date(date).toLocaleDateString(undefined, options);
       }
     },
   }
 </script>