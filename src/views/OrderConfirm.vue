<template>
  <div>
    <nav-header></nav-header>
    <nav-bread>
      <span>Order Confirm</span>
    </nav-bread>
    <div class="container">
      <div class="checkout-order">
        <!-- process step -->
        <div class="check-step">
          <ul>
            <li class="cur"><span>Confirm</span> address</li>
            <li class="cur"><span>View your</span> order</li>
            <li><span>Make</span> payment</li>
            <li><span>Order</span> confirmation</li>
          </ul>
        </div>

        <!-- order list -->
        <div class="page-title-normal checkout-title">
          <h2><span>Order content</span></h2>
        </div>
        <div class="item-list-wrap confirm-item-list-wrap">
          <div class="cart-item order-item">
            <div class="cart-item-head">
              <ul>
                <li>Order contents</li>
                <li>Price</li>
                <li>Quantity</li>
                <li>Subtotal</li>
              </ul>
            </div>
            <ul class="cart-item-list">
              <li v-for="item in cartList" v-if="item.checked" :key="item.﻿productId">
                <div class="cart-tab-1">
                  <div class="cart-item-pic">
                    <img :src="'/static/'+item.﻿productImage" alt="XX">
                  </div>
                  <div class="cart-item-title">
                    <div class="item-name" v-text="item.﻿productName">XX</div>

                  </div>
                </div>
                <div class="cart-tab-2">
                  <div class="item-price">{{item.﻿salePrice | currency('¥')}}</div>
                </div>
                <div class="cart-tab-3">
                  <div class="item-quantity">
                    <div class="select-self">
                      <div class="select-self-area">
                        <span class="select-ipt">×{{item.﻿productNum}}</span>
                      </div>
                    </div>
                    <div class="item-stock item-stock-no">In Stock</div>
                  </div>
                </div>
                <div class="cart-tab-4">
                  <div class="item-price-total">{{item.﻿salePrice*item.﻿productNum | currency('¥')}}</div>
                </div>
              </li>
            </ul>
          </div>
        </div>

        <!-- Price count -->
        <div class="price-count-wrap">
          <div class="price-count">
            <ul>
              <li>
                <span>Item subtotal:</span>
                <span>{{itemSubtotal | currency('¥')}}</span>
              </li>
              <li>
                <span>Shipping:</span>
                <span>{{shipping | currency('¥')}}</span>
              </li>
              <li>
                <span>Discount:</span>
                <span>{{discount | currency('¥')}}</span>
              </li>
              <li>
                <span>Tax:</span>
                <span>{{tax | currency('¥')}}</span>
              </li>
              <li class="order-total-price">
                <span>Order total:</span>
                <span>{{orderTotal | currency('¥')}}</span>
              </li>
            </ul>
          </div>
        </div>

        <div class="order-foot-wrap">
          <div class="prev-btn-wrap">
            <a class="btn btn--m" @click="previous">Previous</a>
          </div>
          <div class="next-btn-wrap">
            <button class="btn btn--m btn--red" @click="payment">Proceed to payment</button>
          </div>
        </div>
      </div>
    </div>
    <nav-footer></nav-footer>
  </div>
</template>
<script>
  import NavHeader from '@/components/NavHeader.vue'
  import NavFooter from '@/components/NavFooter.vue'
  import NavBread from '@/components/NavBread.vue'
  import {currency} from '../util/currency'
  export default{
    data(){
      return{
        shipping: 100,
        discount: 200,
        tax: 400,
        itemSubtotal:0,
        orderTotal: 0,
        cartList: []
      }
    },
    mounted() {
      this.getCartList()
    },
    methods: {
      payment() {
        this.axios.post('/users/payment', {
          addressId: this.$route.query.addressId,
          shipping: this.shipping,
          discount: this.discount,
          tax: this.tax,
          orderTotal: this.orderTotal
        }).then(response => {
          let res = response.data
          if (res.status === '0') {
            this.$router.push({
              name: 'OrderSuccess',
              params: {
                orderId: res.result.orderId,
                orderTotal: res.result.orderTotal
              }
            })
          }
        })
      },
      previous() {
        this.$router.go(-1)
      },
      getCartList() {
        this.axios.get('/users/cartList').then((response) => {
          let res = response.data
          if(res.status === '0') {
            this.cartList = res.result.list
            this.cartList.forEach(item => {
              if(item.checked) this.itemSubtotal += item.﻿salePrice * item.﻿productNum
            })
            this.orderTotal = this.itemSubtotal + this.shipping - this.discount + this.tax
          }
        })
      }
    },
    filters: {
      currency: currency
    },
    components: {
      NavHeader,
      NavFooter,
      NavBread
    }
  }
</script>
