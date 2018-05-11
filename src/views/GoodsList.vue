<template>
    <div>
      <nav-header></nav-header>
      <nav-bread>
        <span>Goods</span>
      </nav-bread>
      <div class="accessory-result-page accessory-page">
        <div class="container">
          <div class="filter-nav">
            <span class="sortby">Sort by:</span>
            <a href="javascript:void(0)" class="default cur">Default</a>
            <a href="javascript:void(0)" class="price" @click="sortGoods">
              Price
              <svg class="icon-arrow-short" :class="{'sort-up':!sortFlag}">
                <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-arrow-short"></use>
              </svg>
            </a>
            <a href="javascript:void(0)" class="filterby stopPop"
              @click="showFilterPop">Filter by</a>
          </div>
          <div class="accessory-result">
            <!-- filter -->
            <div class="filter stopPop" id="filter"
              :class="{'filterby-show':filterBy}">
              <dl class="filter-price">
                <dt>Price:</dt>
                <dd>
                  <a href="javascript:void(0)"
                    :class="{'cur':priceChecked==='all'}"
                    @click="priceChecked='all'">All</a>
                </dd>
                <dd
                  v-for="(price,index) in priceFilter"
                  :key="index"
                  @click="setPriceFilter(index)">
                  <a href="javascript:void(0)" :class="{'cur':priceChecked===index}">{{price.startPrice}} - {{price.endPrice}}</a>
                </dd>
              </dl>
            </div>

            <!-- search result accessories list -->
            <div class="accessory-list-wrap">
              <div class="accessory-list col-4">
                <ul>
                  <li
                    v-for="item in goodsList"
                    :key="item.productId">
                    <div class="pic">
                      <a href="#"><img v-lazy="'/static/'+item.productImage" alt=""></a>
                    </div>
                    <div class="main">
                      <div class="name">{{item.productName}}</div>
                      <div class="price">{{item.﻿salePrice}}</div>
                      <div class="btn-area">
                        <a href="javascript:;" class="btn btn--m" @click="addCart(item.productId)">加入购物车</a>
                      </div>
                    </div>
                  </li>
                </ul>
                <div v-infinite-scroll="loadMore" infinite-scroll-disabled="busy" infinite-scroll-distance="30" class="load-more">
                  <img src="./../assets/loading-spinning-bubbles.svg" v-if="loading"/>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="md-overlay" v-show="overLayFlag" @click="closePop"></div>
      <modal :mdShow="mdShowCart" @close="closeModal">
        <p slot="message">
          <svg class="icon-status-ok">
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#icon-status-ok"></use>
          </svg>
          <span>加入购物车成功！</span>
        </p>
        <div slot="btnGroup">
          <a href="javascript:;" class="btn btn--m" @click="mdShowCart=false">继续购物</a>
          <router-link class="btn btn--m" to="/cart">查看购物车</router-link>
        </div>
      </modal>
      <modal :mdShow="mdShow" @close="closeModal">
        <p slot="message">
          请先登录。否则无法加入到购物车！
        </p>
        <div slot="btnGroup">
          <a href="javascript:;" class="btn btn--m" @click="mdShow=false">关闭</a>
        </div>
      </modal>
      <nav-footer></nav-footer>
    </div>
</template>
<script>
  import NavHeader from '@/components/NavHeader.vue'
  import NavFooter from '@/components/NavFooter.vue'
  import NavBread from '@/components/NavBread.vue'
  import Modal from '@/components/Modal.vue'
  export default{
    data(){
      return {
        goodsList: [],
        priceFilter: [
          {
            startPrice: '0.00',
            endPrice: '500.00'
          },
          {
            startPrice: '500.00',
            endPrice: '1000.00'
          },
          {
            startPrice: '1000.00',
            endPrice: '5000.00'
          }
        ],
        priceChecked: 'all',
        filterBy: false,
        overLayFlag: false,
        sortFlag: true,
        page: 1,
        pageSize: 8,
        busy: true,
        loading: false,
        mdShow: false,
        mdShowCart: false
      }
    },
    methods: {
      getGoodsList(flag) {
        let param = {
          page: this.page,
          pageSize: this.pageSize,
          sort: this.sortFlag?1:-1,
          priceLevel: this.priceChecked
        }
        this.loading = true
        this.axios.get("/goods/list",{
          params: param
        }).then(res => {
          var res = res.data
          this.loading  = false
          if (res.status === '0') {
            if (flag) {
              this.goodsList = this.goodsList.concat(res.result.list)
              if (res.result.count < this.pageSize) {
                this.busy = true
              } else {
                this.busy = false
              }
            } else {
              this.goodsList = res.result.list
              this.busy = false
            }
          } else {
            this.goodsList = []
          }
        })
      },
      showFilterPop() {
        this.filterBy = true
        this.overLayFlag = true
      },
      closePop() {
        this.filterBy = false
        this.overLayFlag = false
      },
      setPriceFilter(index) {
        this.priceChecked = index
        this.closePop()
        this.page = 1
        this.getGoodsList()
      },
      sortGoods() {
        this.sortFlag = !this.sortFlag
        this.page = 1
        this.getGoodsList()
      },
      loadMore() {
        console.log(1)
        this.busy = true
        setTimeout(() => {
          this.page++
          this.getGoodsList(true)
        }, 500);
      },
      addCart(productId) {
        this.axios.post("/goods/addCart", {
          productId: productId
        }).then(response => {
          let res = response.data;
          if (res.success) this.mdShowCart = true
          else this.mdShow = true
          this.$store.commit('updateCartCount', 1)
        })
      },
      closeModal() {
        this.mdShow = false
        this.mdShowCart = false
      }
    },
    mounted () {
      this.getGoodsList()
    },
    components: {
      NavHeader,
      NavFooter,
      NavBread,
      Modal
    }
  }
</script>
