<template>
  <div class="flex flex-column align-start align-self-start justify-start">
    <h1 style = text-align:center>Welcome {{ $nuxt.$fire.auth.currentUser.email }} to E-vacuation</h1>
    <v-carousel height="800px">
    <v-carousel-item
      v-for="(item,i) in items"
      :key="i"
      :src="item.src"
      reverse-transition="fade-transition"
      transition="fade-transition"
    ></v-carousel-item>
  </v-carousel>
  </div>
</template>

<script>

export default {
  data () {
      return {
        items: [
          {
            src: 'https://www.gannett-cdn.com/-mm-/268cf849c0fbfcd7322de9ffa2c4504257305dd3/c=0-343-3500-2317/local/-/media/USATODAY/USATODAY/2013/11/08/1383900301000-AFP-524452087.jpg',
          },
          {
            src: 'https://www.asianews.it/files/img/FILIPPINE_-_0208_-_Devastante_incendio_nella_periferia_di_Manila._Evacuate_15_mila_persone.jpg',
          },
          {
            src: 'https://gdb.voanews.com/79C8C21A-BA0F-4D75-97B3-07470AC9B71D_cx0_cy2_cw0_w1200_r1.jpg',
          },
          {
            src: 'https://mycebu.ph/wp-content/uploads/2016/10/colon-flood.jpg',
          },
          {
            src: 'https://gdb.voanews.com/1F027C00-19BD-4E49-A7AC-E5E8D9F2CDA8_w1200_r1.jpg',
          },
          {
            src: 'http://assets.rappler.com/612F469A6EA84F6BAE882D2B94A4B421/img/44D56EFEF0DA47B8B5782AC7C2C4C8D8/afp-file-20041228-meulaboh-aceh-tsunami_44D56EFEF0DA47B8B5782AC7C2C4C8D8.jpg',
          },
        ],
      }
    },
  layout: 'default',
}
</script>


