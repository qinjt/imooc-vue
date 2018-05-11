<template>
  <div>
    <nav-header></nav-header>
    <nav-bread>
      <span>Cart</span>
    </nav-bread>
    <div class="container">
      <div class="cart">
        <div class="page-title-normal">
          <h2 class="page-title-h2"><span>My Cart</span></h2>
        </div>
        <div class="item-list-wrap">
          <div class="cart-item">
            <div class="cart-item-head">
              <ul>
                <li>Items</li>
                <li>Price</li>
                <li>Quantity</li>
                <li>Subtotal</li>
                <li>Edit</li>
              </ul>
            </div>
            <ul class="cart-item-list">
              <li v-for="item in cartList" :key="item.productId">
                <div class="cart-tab-1">
                  <div class="cart-item-check">
                    <a class="checkbox-btn item-check-btn"
                       :class="{'check': item.checked}"
                       @click="editCart('check', item)">
                      <svg class="icon icon-ok">
                        <use xlink:href="#icon-ok"></use>
                      </svg>
                    </a>
                  </div>
                  <div class="cart-item-pic">
                    <img :src="'/static/'+item.productImage" :alt="item.productName">
                  </div>
                  <div class="cart-item-title">
                    <div class="item-name" v-text="item.productName">XX</div>
                  </div>
                </div>
                <div class="cart-tab-2">
                  <div class="item-price">{{item.salePrice | currency('¥')}}</div>
                </div>
                <div class="cart-tab-3">
                  <div class="item-quantity">
                    <div class="select-self select-self-open">
                      <div class="select-self-area">
                        <a class="input-sub" @click="editCart('minu', item)">-</a>
                        <span class="select-ipt" v-text="item.productNum"></span>
                        <a class="input-add" @click="editCart('add', item)">+</a>
                      </div>
                    </div>
                  </div>
                </div>
                <div class="cart-tab-4">
                  <div class="item-price-total">{{(item.salePrice * item.productNum) | currency('¥')}}</div>
                </div>
                <div class="cart-tab-5">
                  <div class="cart-item-opration">
                    <a href="javascript:;" class="item-edit-btn" @click="delCartConfirm(item)">
                      <svg class="icon icon-del">
                        <use xlink:href="#icon-del"></use>
                      </svg>
                    </a>
                  </div>
                </div>
              </li>
            </ul>
          </div>
        </div>
        <div class="cart-foot-wrap">
          <div class="cart-foot-inner">
            <div class="cart-foot-l">
              <div class="item-all-check">
                <a @click="toggleCheckAll">
                  <span class="checkbox-btn item-check-btn" :class="{'check': checkAllFlag}">
                      <svg class="icon icon-ok"><use xlink:href="#icon-ok"/></svg>
                  </span>
                  <span>Select all</span>
                </a>
              </div>
            </div>
            <div class="cart-foot-r">
              <div class="item-total">
                Item total: <span class="total-price">{{totalPrice | currency('¥')}}</span>
              </div>
              <div class="btn-wrap">
                <a class="btn btn--red" :class="{'btn--dis': !checkedCount}" @click="checkout">Checkout</a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <modal :mdShow="modalConfirm" @close="closeModal">
      <p slot="message">你确认要删除此条数据吗？</p>
      <div slot="btnGroup">
        <a href="javascript:;" class="btn btn--m" @click="delGood">确认</a>
        <a href="javascript:;" class="btn btn--m" @click="modalConfirm=false">取消</a>
      </div>
    </modal>
    <nav-footer></nav-footer>
  </div>
</template>
<style>
  .input-sub,.input-add{
    min-width: 40px;
    height: 100%;
    border: 0;
    color: #605F5F;
    text-align: center;
    font-size: 16px;
    overflow: hidden;
    display: inline-block;
    background: #f0f0f0;
  }
  .item-quantity .select-self-area{
    background:none;
    border: 1px solid #f0f0f0;
  }
  .item-quantity .select-self-area .select-ipt{
    display: inline-block;
    padding:0 3px;
    width: 30px;
    min-width: 30px;
    text-align: center;
  }
</style>
<script>
  import NavHeader from '@/components/NavHeader.vue'
  import NavFooter from '@/components/NavFooter.vue'
  import NavBread from '@/components/NavBread.vue'
  import Modal from '@/components/Modal.vue'
  import {currency} from '../util/currency'
  export default{
    data(){
      return{
        cartList: [],
        modalConfirm: false,
        delItem: {}
      }
    },
    computed: {
      checkAllFlag() {
        return this.checkedCount === this.cartList.length
      },
      checkedCount() {
        let i = 0
        this.cartList.forEach(item => {
          if (item.checked) i++
        })
        return i
      },
      totalPrice() {
        let totalMoney = 0
        this.cartList.forEach(item => {
          if (item.checked) {
            totalMoney += parseInt(item.productNum) * parseFloat(item.salePrice)
          }
        })
        return totalMoney
      }
    },
    filters: {
      currency: currency
    },
    mounted() {
      this.getCartList()
    },
    methods: {
      getCartList() {
        this.axios.get('/users/cartList').then((response) => {
          let res = response.data
          this.cartList = res.result.list
        })
      },
      closeModal() {
        this.modalConfirm = false
      },
      delCartConfirm(item) {
        this.modalConfirm = true
        this.delItem = item
      },
      delGood() {
        this.axios.post('/users/cartDel', {
          productId: this.delItem.productId
        }).then((response) => {
          let res = response.data
          this.closeModal()
          if (res.status === '0') {
            this.cartList.forEach((item,index) => {
              if (item.productId === this.delItem.productId) {
                this.cartList.splice(index, 1)
                return
              }
            })
            this.$store.commit('updateCartCount', -this.delItem.productNum)
          }
        })
      },
      editCart(flag, item) {
        this.delItem = item
        this.productId = item.productId
        if (flag === 'add') {
          item.productNum ++
        } else if (flag === 'minu') {
          item.productNum --
          if (item.productNum <= 0) {
            this.delGood()
          }
        } else {
          item.checked = !item.checked
        }
        this.axios.post('/users/cartEdit', {
          productId: item.productId,
          productNum: item.productNum,
          checked: item.checked
        }).then((response) => {
          let res = response.data
          let num = 0
          if(flag === 'add') {
            num = 1
          } else if(flag === 'minu') {
            num = -1
          }
          this.$store.commit('updateCartCount', num)
        })
      },
      toggleCheckAll() {
        let flag = !this.checkAllFlag
        this.cartList.forEach(item => {
          item.checked = flag
        })
        this.axios.post('/users/cartCheckAll', {
          checkAll: flag
        }).then(response => {
          let res = response.data
        })
      },
      checkout() {
        if (!this.checkedCount) return
        this.$router.push({
          name: 'Address'
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
