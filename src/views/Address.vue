<template>
    <div>
      <nav-header></nav-header>
      <nav-bread>
        <span>Address</span>
      </nav-bread>
      <div class="checkout-page">
        <div class="container">
          <div class="checkout-addr">
            <div class="page-title-normal">
              <h2 class="page-title-h2"><span>check out</span></h2>
            </div>
            <!-- process step -->
            <div class="check-step">
              <ul>
                <li class="cur"><span>Confirm</span> address</li>
                <li><span>View your</span> order</li>
                <li><span>Make</span> payment</li>
                <li><span>Order</span> confirmation</li>
              </ul>
            </div>

            <!-- address list -->
            <div class="page-title-normal checkout-title">
              <h2><span>Shipping address</span></h2>
            </div>
            <div class="addr-list-wrap">
              <div class="addr-list">
                <ul>
                  <li v-for="(item,index) in addressListFilter"
                      :keys="item.﻿addressId"
                      :class="{'check':checkIndex===index}"
                      @click="checkIndex=index;selectAddressId=item.addressId">
                    <dl>
                      <dt v-text="item.﻿userName"></dt>
                      <dd class="address" v-text="item.﻿streetName"></dd>
                      <dd class="tel" v-text="item.﻿tel"></dd>
                    </dl>
                    <div class="addr-opration addr-del">
                      <a href="javascript:;" class="addr-del-btn" @click="delAddress(item.addressId)">
                        <svg class="icon icon-del"><use xlink:href="#icon-del"></use></svg>
                      </a>
                    </div>
                    <div v-if="!item.﻿isDefault" class="addr-opration addr-set-default" @click="setDefaultAddress(item.addressId)">
                      <a href="javascript:;" class="addr-set-default-btn"><i>Set default</i></a>
                    </div>
                    <div v-else class="addr-opration addr-default">Default address</div>
                  </li>
                  <li class="addr-new">
                    <div class="add-new-inner">
                      <i class="icon-add">
                        <svg class="icon icon-add"><use xlink:href="#icon-add"></use></svg>
                      </i>
                      <p>Add new address</p>
                    </div>
                  </li>
                </ul>
              </div>

              <div class="shipping-addr-more">
                <a class="addr-more-btn up-down-btn" href="javascript:;" @click="expand" :class="{'open': limit>3}">
                  more
                  <i class="i-up-down">
                    <i class="i-up-down-l"></i>
                    <i class="i-up-down-r"></i>
                  </i>
                </a>
              </div>
            </div>

            <!-- shipping method-->
            <div class="page-title-normal checkout-title">
              <h2><span>Shipping method</span></h2>
            </div>
            <div class="shipping-method-wrap">
              <div class="shipping-method">
                <ul>
                  <li class="check">
                    <div class="name">Standard shipping</div>
                    <div class="price">Free</div>
                    <div class="shipping-tips">
                      <p>Once shipped，Order should arrive in the destination in 1-7 business days</p>
                    </div>
                  </li>
                </ul>
              </div>
            </div>
            <div class="next-btn-wrap">
              <router-link class="btn btn--m btn--red" :to="{path:'/orderConfirm',query:{'addressId':selectAddressId}}">Next</router-link>
            </div>
          </div>
        </div>
      </div>
      <modal :mdShow="isMdShow" @close="closeModal">
        <p slot="message">您是否确认删除此条地址？</p>
        <div slot="btnGroup">
          <a href="javascript:;" class="btn btn--m" @click="confirmDelAddress">确认</a>
          <a href="javascript:;" class="btn btn--m" @click="isMdShow=false">取消</a>
        </div>
      </modal>
      <nav-footer></nav-footer>
    </div>
</template>
<style>
</style>
<script>
  import NavHeader from '@/components/NavHeader.vue'
  import NavFooter from '@/components/NavFooter.vue'
  import NavBread from '@/components/NavBread.vue'
  import Modal from '@/components/Modal.vue'
  
  const addressLimit = 3
  
  export default{
    data(){
      return{
        limit: addressLimit,
        addressList: [],
        selectAddressId: '',
        checkIndex: 0,
        isMdShow: false,
        delAddressId: ''
      }
    },
    computed: {
      addressListFilter() {
        return this.addressList.slice(0, this.limit)
      }
    },
    mounted() {
      this.getAddressList()
    },
    methods: {
      getAddressList() {
        this.axios.get('/users/addressList').then(response => {
          let res = response.data
          if (res.status === '0') {
            this.addressList = res.result.list
            this.addressList.forEach(item => {
              if(item.﻿isDefault) this.selectAddressId = item.addressId
            })
          }
        })
      },
      expand() {
        if (this.limit === addressLimit) this.limit = this.addressList.length
        else this.limit = addressLimit
      },
      delAddress(addressId) {
        this.isMdShow = true
        this.delAddressId = addressId
      },
      closeModal() {
        this.isMdShow = false
      },
      confirmDelAddress() {
        this.axios.post('/users/delAddress', {
          addressId: this.delAddressId
        }).then((response) => {
          let res = response.data
          if (res.status === '0') {
            this.isMdShow = false
            this.addressList.forEach((item, index) => {
              if(item.addressId === this.delAddressId) {
                this.addressList.splice(index, 1)
              }
            })
          }
        })
      },
      setDefaultAddress(addressId){
        this.axios.post('/users/setDefault', {
          addressId: addressId
        }).then((response) => {
          let res = response.data
          if (res.status === '0') {
            this.addressList.forEach(item => {
              if (item.addressId === addressId) item.﻿isDefault = true
              else item.﻿isDefault = false
            });
          }
        })
      }
    },
    components: {
      NavHeader,
      NavFooter,
      NavBread,
      Modal
    }
  }
</script>
