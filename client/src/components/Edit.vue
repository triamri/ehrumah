<template>
  <div class="row">

        <sidebar></sidebar>
        <!-- /.col-lg-3 -->

        <div class="col-lg-9">

          <div class="row">
          <div class="card col-lg-6 col-md-4 mb-4">
            <a href="#"><img class="card-img-top" v-bind:src="image" alt=""></a>
          </div>
        </div>
        <div class="col-lg-6 col-md-6 mb-4">
          <div class="card h-100">
            <div class="card-body">
              <input v-model="name_item" placeholder="Name Item">
              <input v-model="address" placeholder="Address">
              <input v-model="type" placeholder="type">
              <input v-model="luas" placeholder="Luas Tanah">
              <input v-model="sale" placeholder="Sale">
              <div class="col-md-10 offset-md-1">
              <label>
                AutoComplete
                <GmapAutocomplete @place_changed="setPlace">
                </GmapAutocomplete>
              </label>

              <gmap-map :center="center" :zoom="zoom" style="width: 100%; height: 600px">
                <gmap-marker v-if="marker" :clickable="true" :position="marker.position" :draggable="true" @drag="markerChanged"> </gmap-marker>
              </gmap-map>
              </div>
              <input type="file" id="file">
              <button type="button" @click="updateData">Save</button>
            </div>
          </div>
        </div>      
      </div>
      <!-- /.row -->

    </div>
</template>

<script>

import axios from 'axios'

import Sidebar from './Sidebar'

export default {
  components: {
    Sidebar
  },
  data () {
    return {
      msg: 'Edit data',
      image: null,
      name_item: null,
      address: null,
      type: null,
      luas: null,
      sale: null,
      center: {},
      zoom: 10,
      marker: null 
    }
  },
  created () {
    axios.get(`http://localhost:3000/api/types/detail/${this.$route.params.id}`)
    .then((respone)=>{
      console.log(respone.data.data.name_item)
      this.name_item = respone.data.data.name_item;
      this.address = respone.data.data.address;
      this.type = respone.data.data.specific.type;
      this.luas = respone.data.data.specific.luas;
      this.sale = respone.data.data.sale;
      this.image = respone.data.data.image;
      this.marker = {
        position: {
          lat: Number(respone.data.data.lat),
          lng: Number(respone.data.data.lng)
        }
      }
      this.center = {
        lat: Number(respone.data.data.lat),
        lng: Number(respone.data.data.lng)
      }
        
      // this.userId = respone.data.data.userId;
    })
    .catch((error)=>{
      console.log(error);
    })
  },
  methods:{
    updateData(){

      if(document.getElementById('file').files[0]){

        const data = new FormData();
        data.append('image',document.getElementById('file').files[0]);
        data.append('name_item',this.name_item);
        data.append('address',this.address);
        data.append('type',this.type);
        data.append('luas',this.luas);
        data.append('sale',this.sale);
        data.append('lat',this.marker.position.lat);
        data.append('lng',this.marker.position.lng);

        axios.put(`http://localhost:3000/api/types/update/${this.$route.params.id}`, data,{
          headers:{
            token: localStorage.getItem('authUser')
          }
        })
        .then((respone)=>{
          // console.log(respone.data)
          this.$router.push('/about/sale');
        })
        .catch((error)=>{
          console.log(error);
        })

      }else{

        axios.put(`http://localhost:3000/api/types/nofoto/${this.$route.params.id}`,{
          name_item: this.name_item,
          address: this.address,
          type: this.type,
          luas: this.luas,
          sale: this.sale,
          lat: this.marker.position.lat,
          lng: this.marker.position.lng
        },{
          headers:{
            token: localStorage.getItem('authUser')
          }
        })
        .then((respone)=>{
          // console.log(respone.data)
          this.$router.push('/about/sale');
        })
        .catch((error)=>{
          console.log(error);
        })

      }
    },
    setPlace: function(place) {
      this.zoom = 12;
      this.center = {
        lat: place.geometry.location.lat(),
        lng: place.geometry.location.lng()
      };
      this.marker = {
        position: {
          lat: place.geometry.location.lat(),
          lng: place.geometry.location.lng()
        }
      };
    },
    markerChanged(xxx) {
      this.marker = {
        position: {
          lat: xxx.latLng.lat(),
          lng: xxx.latLng.lng()
        }
      };
    }
  }
}
</script>