<template>
  <q-page padding>
    <div class="q-mb-xl">
      <q-carousel
          animated
          v-model="slide"
          navigation
          infinite
          autoplay
          arrows
          swipeable
          transition-prev="slide-right"
          transition-next="slide-left"
          >
          <q-carousel-slide :name="1" img-src="https://cdn.luxe.digital/media/2020/05/07203610/best-coffee-beans-luxe-digital.jpg" />
          <q-carousel-slide :name="2" img-src="https://www.bravotangocoffee.com/wp-content/uploads/2020/12/brewmethods.jpg" />
          <q-carousel-slide :name="3" img-src="https://thehealthytreehouse.com/wp-content/uploads/2021/04/Types-of-Coffee-Drinks-.jpg" />
          <q-carousel-slide :name="4" img-src="https://assets.pikiran-rakyat.com/crop/0x0:0x0/x/photo/2021/01/21/2042538411.jpg" />
      </q-carousel>
    </div>
      <div class="row q-col-gutter-md">
      <div class="col-md-3 col-xs-12" v-for="(coffee, i) in data" :key="i">
        <q-card>
        <q-img
          :src="`http://localhost:5000/${coffee.image}`"
          :ratio="16/9"
          />

        <q-card-section>
        <q-btn
        fab
        color="primary"
        icon="add_shopping_cart"
        class="absolute"
        unelevated
        style="top: 0; right: 12px; transform: translateY(-50%);"
        />

        <div class="row no-wrap items-center">
          <div class="col text-h6 ellipsis">
            {{ coffee.namaCoffee }}
          </div>
        </div>

          <q-rating v-model="coffee.rating" readonly color="yellow-5" size="32px" />
        </q-card-section>

        <q-card-section class="q-pt-none">
          <div class="text-subtitle1">
            Rp {{ coffee.harga }},-
          </div>
          <div @click="coffee.expanded = !coffee.expanded" class="text-caption text-grey cursor-pointer">
            Tampilkan Deskripsi
          </div>
          <q-slide-transition>
            <div v-show="coffee.expanded">
              <div class="text-grey text-caption">
                {{ coffee.deskripsi }}
              </div>
            </div>
          </q-slide-transition>
        </q-card-section>

      <q-card-actions>
        <q-btn unelevated @click="openDetail(coffee)" class="full-width" color="primary">
          Order Now
        </q-btn>
      </q-card-actions>
      </q-card>
        </div>
      </div>
    <q-dialog v-model="dialog" v-if="dialog" position="bottom">
      <q-card style="width: 500px">
        <q-card-section>
          <div class="text-h6">Detail Pemesanan</div>
        </q-card-section>
        <q-card-section style="max-height: 50vh;" class="scroll">
            {{ activeData.namaCoffee }} - Rp {{ activeData.harga }},-
            <q-form class="q-mt-md">
                <q-input filled type="number" class="q-mb-md" v-model="jumlah" label="Masukan Jumlah"/>
                Rp {{ total }},-
                <q-file color="teal" class="q-mt-md" filled v-model="image" label="Upload Bukti Pembayaran">
                    <template v-slot:prepend>
                     <q-icon name="cloud_upload" />
                    </template>
                </q-file>
            </q-form>
        </q-card-section>
        <q-card-actions align="right">
            <q-btn flat label="Batal" v-close-popup/>
            <q-btn color="primary" @click="prosesTransaksi" label="Proses"/>
        </q-card-actions>
      </q-card>
    </q-dialog>
    </q-page>
</template>
<script>
export default {
  data () {
    return {
      slide: 1,
      stars: 4,
      dialog: false,
      image: null,
      data: [],
      activeData: null,
      jumlah: 1,
      namaCoffe: null
    }
  },
  created () {
    this.getData()
  },
  methods: {
    getData () {
      this.$axios.get('coffee/getall')
        .then(res => {
          if (res.data.sukses) {
            this.data = res.data.data.map(coffee => {
              return Object.assign(coffee, {
                expanded: false
              })
            })
          } else {
            this.$showNotif(res.data.pesan, 'negative')
          }
        })
    },
    openDetail (data) {
      this.activeData = data
      this.dialog = true
    },
    prosesTransaksi () {
      const formData = new FormData()
      formData.append('image', this.image)
      formData.append('data', JSON.stringify({
        idUser: this.$q.localStorage.getItem('dataUser')._id,
        idCoffee: this.activeData._id,
        harga: this.activeData.harga,
        jumlah: this.jumlah,
        total: this.total
      }))
      this.$axios.post('order/insert', formData)
        .then(res => {
          if (res.data.sukses) {
            this.$showNotif(res.data.pesan, 'positive')
            this.dialog = false
            this.image = null
          } else {
            this.$showNotif(res.data.pesan, 'negative')
          }
        })
    }
  },
  computed: {
    total () {
      return this.activeData.harga * this.jumlah
    }
  }
}
</script>
