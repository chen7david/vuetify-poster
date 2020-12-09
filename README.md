# vuetify-poster

### props 
- :item="item",
- :src="item.url",
- width="168",
- ratio="2/3", 
- spinner: 'red',
- iconSize="23",
- zIndex="1",


### poster.vue
```vue
<template>
    <v-hover v-show="isVisible" absolute v-slot="{ hover }" >
        <v-card tile :width="width" color="transparent" elevation="0">
            <v-card tile dark elevation="0">
                <!-- CARC TOP TOOLBAR -->
                <v-toolbar dense absolute class="flex-grow-0" color="transparent" elevation="0">
                    <v-icon v-if="isSelected" :size="iconsize">mdi-checkbox-marked-circle</v-icon>
                </v-toolbar>
                <!-- CARD IMAGE -->
                <v-lazy :options="{threshold: 0.5}">
                    <v-img :src="src" :aspect-ratio="aspect" @error="error" @load="done">
                        <!-- CARD IMAGE LOAD SPINNER -->
                        <v-row class="fill-height" align="center" justify="center">
                            <v-progress-circular 
                                v-if="imgLoading" 
                                indeterminate 
                                :color="spinner_color"
                            ></v-progress-circular>
                        </v-row>
                    </v-img>
                </v-lazy>
                <!-- CARD OVERLAY -->
                <v-overlay :z-index="ZIndex" absolute :value="hover && !isLoading && !imgLoading">
                    <v-sheet :width="width" :height="height" color="transparent" class="d-flex flex-column">
                        <!-- TOP TOOLBAR -->
                        <v-toolbar dense class="flex-grow-0" color="transparent" elevation="0">
                            <v-icon v-if="!isSelected" :size="iconsize" @click="select" @mouseover="hoverSelect = true" @mouseleave="hoverSelect = false">
                                mdi-checkbox-blank-circle{{outline}}
                            </v-icon>
                            <v-icon v-if="isSelected" :size="iconsize" @click="deselect">
                                mdi-checkbox-marked-circle
                            </v-icon>
                            <v-spacer></v-spacer>
                            <slot name="tr" v-bind:prop="{iconsize, load}"/>
                        </v-toolbar>
                        <!-- MID-SECTION -->
                        <v-row justify="center" class="flex-grow-1">
                            <slot name="body" v-bind:prop="{iconsize, load}"/>
                        </v-row>
                        <!-- BOTTOM TOOLBAR -->
                        <v-toolbar dense class="flex-grow-0" color="transparent" elevation="0">
                            <slot name="bl" v-bind:prop="{iconsize, load}"/>
                            <v-spacer></v-spacer>
                            <slot name="br" v-bind:prop="{iconsize, load}"/>
                        </v-toolbar>
                    </v-sheet>
                </v-overlay>
            </v-card>
            <v-card-subtitle class="text-left pa-0">
                <slot name="footer" v-bind:prop="{iconsize, load}"/>
            </v-card-subtitle>
        </v-card>
    </v-hover>
</template>

<script>
export default {
    name: 'poster',
    props: {
        item: null,
        src: null,
        width: null,
        ratio: null, 
        spinner: null,
        iconSize: null,
        zIndex: null,
    },
    data: () => ({
        isLoading: false,
        isSelected: false,
        isVisible: true,
        hoverPlay: false,
        hoverSelect: false,
        imgDone: false,
        imgError: false,
    }),
    computed: {
        aspect(){ return this.ratio || 2/3 },
        height(){ return this.width/this.aspect },
        imgLoading(){ return !this.imgError && !this.imgDone || this.isLoading },
        spinner_color(){ return this.spinner || 'orange' },
        iconsize(){ return this.iconSize || 20},
        outline(){ return this.hoverSelect ? ``:`-outline`},
        ZIndex(){ return this.zIndex || 1},
    },
    methods: {
        done(){ this.imgDone = true },
        error(){ this.imgError = true },
        select(){
            this.isSelected = true
            this.$emit('select', this.item)
        },
        deselect(){
            this.isSelected = false
            this.$emit('deselect', this.item)
        },
        hide(){ this.isVisible = false },
        show(){ this.isVisible = true },
        async load(func, params = {}){
            this.isLoading = true
            const data = await func(params)
            this.isLoading = false
            return data
        },
    },
}
</script>
```

### componnent.vue
```vue
<template>
  <div class="home">
    <v-app-bar fixed dense v-if="showToolbar">
      <v-btn text tile @click="selectAll">select all</v-btn>
    <v-btn text tile @click="deselectAll">deselect all</v-btn>
    <v-btn text tile @click="hideAll">hide</v-btn>
    <v-btn text tile @click="showAll">show</v-btn>
    <v-btn text tile @click="addAll">run all</v-btn>
    </v-app-bar>
    <v-col cols="12">
      <v-row justify="space-around">
        <poster
          ref="posters"
          v-for="item of items"
          :key="item.id"
          :item="item"
          src="http://image.tmdb.org/t/p/original/6YPzBcMH0aPNTvdXNCDLY0zdE1g.jpg"
          width="180"
          :title="item.name"
          :date="item.date"
          @select="select"
          @deselect="deselect"
          iconSize="23"
          spinner="blue"
        >
          <template v-slot:tr="{prop: {iconsize, load}}">
            <v-icon :size="iconsize" @click="load(add,{item})">
                mdi-plus
            </v-icon>
          </template>

          <template v-slot:body>
            <v-col cols="12" class="text-center" align-self="center">
                <v-icon v-if="!hoverPlay" @mouseover="hoverPlay = true" @mouseleave="hoverPlay = false" size="60">mdi-play-circle-outline</v-icon>
                <v-icon v-if="hoverPlay" @mouseover="hoverPlay = true" @mouseleave="hoverPlay = false"  size="60">mdi-play-circle</v-icon>
            </v-col>
          </template>

          <template v-slot:bl="{prop: {iconsize}}">
            <v-icon :size="iconsize">mdi-pencil</v-icon>
          </template>

          <template v-slot:br="{prop: {iconsize}}">
            <v-icon :size="iconsize">mdi-dots-vertical</v-icon>
          </template>
          <template v-slot:footer>
              <v-tooltip top>
                  <template v-slot:activator="{ on, attrs }">
                      <div v-on="on" v-bind="attrs" class="font-weight-medium text-truncate">{{item.name}}</div>
                  </template>
                  <span>{{item.name}}</span>
              </v-tooltip>
              <div class="blue-grey--text">{{year(item.date)}}</div>
          </template>
        </poster>
      </v-row>
    </v-col>
  </div>
</template>

<script>
// @ is an alias to /src
import poster from '@/components/Poster.vue'

export default {
  name: 'Home',
  components: {
    poster
  },
  data: () => ({
    hoverPlay:false,
    showToolbar: true,
    items: [
      { id: 1, name: 'Aname', date:'2021-09-23'},
      { id: 2, name: 'Bname', date:'2021-09-23'},
      { id: 3, name: 'Cname', date:'2021-09-23'},
      { id: 4, name: 'Dname', date:'2021-09-23'},
    ],
    selected: [],
  }),
  watch: {
    selected(){
      this.showToolbar = this.selected.length > 0
    }
  },
  methods: {
    select(item){
      this.selected.push(item)
      console.log('select',item)
    },
    deselect(item){
      const index = this.selected.indexOf(item)
      this.selected.splice(index, 1)
    },

    async add({item}){
      console.log(item)
      return await this.$http.get('/movies')
    },

    hideAll(){
      this.$refs.posters.filter(e => e.isSelected).map(p => p.hide())
    },

    selectAll(){
      this.$refs.posters.map(p => p.select())
    },

    deselectAll(){
      this.$refs.posters.map(p => p.deselect())
    },

    showAll(){
      this.$refs.posters.filter(e => e.isSelected).map(p => p.show())
    },

    addAll(){
        this.$refs.posters.filter(e => e.isSelected).map(p => p.load(this.add,{item:5}))
    },
    year(date){ return date ? new Date(date).getFullYear().toString() : '' },
  }
}
</script>
```