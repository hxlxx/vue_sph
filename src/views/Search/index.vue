<template>
  <div>
    <TypeNav />
    <div class="main">
      <div class="py-container">
        <!--bread-->
        <div class="bread">
          <ul class="fl sui-breadcrumb">
            <li>
              <a href="#">全部结果</a>
            </li>
          </ul>
          <!-- 面包屑标签 -->
          <ul class="fl sui-tag">
            <li class="with-x" v-if="queryInfo.categoryName">{{ queryInfo.categoryName }}<i @click="removeCategory">×</i></li>
            <li class="with-x" v-if="queryInfo.keyword">{{ queryInfo.keyword }}<i @click="removeKeyWord">×</i></li>
            <li class="with-x" v-if="queryInfo.trademark">{{ queryInfo.trademark.split(':')[1] }}<i @click="removeTradeMark">×</i></li>
            <li class="with-x" v-for="(item, index) in queryInfo.props" :key="index">
              {{ item.split(':')[1] }}<i @click="removeAttrInfo(index)">×</i>
            </li>
          </ul>
        </div>

        <!--selector-->
        <SearchSelector :trademarkList="trademarkList" :attrsList="attrsList" @tradeMarkInfo="tradeMarkInfo" @attrInfo="attrInfo" />

        <!--商品排序-->
        <div class="details clearfix">
          <div class="sui-navbar">
            <div class="navbar-inner filter">
              <ul class="sui-nav">
                <li :class="{ active: isOne }">
                  <a @click="changeOrder('1')"
                    >综合
                    <span v-show="isOne" class="iconfont" :class="{ 'icon-sort-down': isDown, 'icon-sort-up': !isDown }"></span>
                  </a>
                </li>
                <li :class="{ active: isTwo }">
                  <a @click="changeOrder('2')"
                    >价格
                    <span v-show="isTwo" class="iconfont" :class="{ 'icon-sort-down': isDown, 'icon-sort-up': !isDown }"></span>
                  </a>
                </li>
              </ul>
            </div>
          </div>
          <div class="goods-list">
            <ul class="yui3-g">
              <li class="yui3-u-1-5" v-for="item in goodsList" :key="item.id">
                <div class="list-wrap">
                  <div class="p-img">
                    <router-link :to="`/detail/${item.id}`">
                      <img :src="item.defaultImg" />
                    </router-link>
                  </div>
                  <div class="price">
                    <strong>
                      <em>¥</em>
                      <i style="margin-left: 3px">{{ item.price }}</i>
                    </strong>
                  </div>
                  <div class="attr">
                    <a>
                      {{ item.title }}
                    </a>
                  </div>
                  <div class="commit">
                    <i class="command">已有<span>2000</span>人评价</i>
                  </div>
                  <div class="operate">
                    <a href="success-cart.html" target="_blank" class="sui-btn btn-bordered btn-danger">加入购物车</a>
                    <a href="javascript:void(0);" class="sui-btn btn-bordered">收藏</a>
                  </div>
                </div>
              </li>
            </ul>
          </div>
          <!-- 分页器 -->
          <Pagination :pageSize="queryInfo.pageSize" :pageNo="queryInfo.pageNo" :total="total" :continues="5" @getCurPage="getCurPage"></Pagination>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import SearchSelector from './SearchSelector/SearchSelector'
export default {
  name: 'Search',
  data() {
    return {
      queryInfo: {
        category1Id: '',
        category2Id: '',
        category3Id: '',
        categoryName: '',
        keyword: '',
        props: [],
        trademark: '',
        order: '1:desc',
        pageNo: 1,
        pageSize: 5
      }
    }
  },
  components: {
    SearchSelector
  },
  mounted() {
    Object.assign(this.queryInfo, this.$route.params, this.$route.query)
    this.getData()
  },
  computed: {
    ...mapGetters(['goodsList', 'attrsList', 'trademarkList', 'total']),
    isOne() {
      return this.queryInfo.order.indexOf('1') !== -1
    },
    isTwo() {
      return this.queryInfo.order.indexOf('2') !== -1
    },
    isDown() {
      if (this.queryInfo.order.indexOf('desc') !== -1) return true
      else return false
    }
  },
  methods: {
    getData() {
      this.$store.dispatch('getSearchList', this.queryInfo)
      this.queryInfo.category1Id = ''
      this.queryInfo.category2Id = ''
      this.queryInfo.category3Id = ''
    },
    // 删除 query 信息
    removeCategory() {
      this.queryInfo.categoryName = ''
      this.getData()
      // 判断此时$route是否携带params参数
      if (this.$route.params) {
        this.$router.push({ name: 'search', params: this.$route.params })
      }
    },
    // 删除 keyword 信息
    removeKeyWord() {
      this.queryInfo.keyword = ''
      this.getData()
      // 通过全局事件总线通知Header组件清空搜索框
      this.$bus.$emit('clear')
      // 判断此时$route是否携带query参数
      if (this.$route.query) {
        this.$router.push({ name: 'search', query: this.$route.query })
      }
    },
    // 自定义事件获取子组件传递的品牌信息
    tradeMarkInfo(trademark) {
      this.queryInfo.trademark = `${trademark.tmId}:${trademark.tmName}`
      this.getData()
    },
    // 删除 trademark 信息
    removeTradeMark() {
      this.queryInfo.trademark = ''
      this.getData()
    },
    // 自定义事件获取子组件传递的商品参数信息
    attrInfo(props) {
      if (this.queryInfo.props.indexOf(props) === -1) {
        this.queryInfo.props.push(props)
      }
      this.getData()
    },
    // 删除商品参数信息
    removeAttrInfo(index) {
      this.queryInfo.props.splice(index, 1)
      this.getData()
    },
    changeOrder(flag) {
      const originIndex = this.queryInfo.order.split(':')[0]
      const originOrder = this.queryInfo.order.split(':')[1]
      let newOrder = ''
      if (originIndex === flag) {
        newOrder = `${flag}:${originOrder === 'desc' ? 'asc' : 'desc'}`
      } else {
        newOrder = `${flag}:desc`
      }
      this.queryInfo.order = newOrder
      this.getData()
    },
    // 获取当前页码并重新请求数据
    getCurPage(pageNo) {
      this.queryInfo.pageNo = pageNo
      this.getData()
    }
  },
  watch: {
    $route: {
      handler() {
        Object.assign(this.queryInfo, this.$route.params, this.$route.query)
        this.getData()
      }
    }
  }
}
</script>

<style lang="less" scoped>
.main {
  margin: 10px 0;

  .py-container {
    width: 1200px;
    margin: 0 auto;

    .bread {
      margin-bottom: 5px;
      overflow: hidden;

      .sui-breadcrumb {
        padding: 3px 15px;
        margin: 0;
        font-weight: 400;
        border-radius: 3px;
        float: left;

        li {
          display: inline-block;
          line-height: 18px;

          a {
            color: #666;
            text-decoration: none;

            &:hover {
              color: #4cb9fc;
            }
          }
        }
      }

      .sui-tag {
        margin-top: -5px;
        list-style: none;
        font-size: 0;
        line-height: 0;
        padding: 5px 0 0;
        margin-bottom: 18px;
        float: left;

        .with-x {
          font-size: 12px;
          margin: 0 5px 5px 0;
          display: inline-block;
          overflow: hidden;
          color: #000;
          background: #f7f7f7;
          padding: 0 7px;
          height: 20px;
          line-height: 20px;
          border: 1px solid #dedede;
          white-space: nowrap;
          transition: color 400ms;
          cursor: pointer;

          i {
            margin-left: 10px;
            cursor: pointer;
            font: 400 14px tahoma;
            display: inline-block;
            height: 100%;
            vertical-align: middle;
          }

          &:hover {
            color: #28a3ef;
          }
        }
      }
    }

    .details {
      margin-bottom: 5px;

      .sui-navbar {
        overflow: visible;
        margin-bottom: 0;

        .filter {
          min-height: 40px;
          padding-right: 20px;
          background: #fbfbfb;
          border: 1px solid #e2e2e2;
          padding-left: 0;
          border-radius: 0;
          box-shadow: 0 1px 4px rgba(0, 0, 0, 0.065);

          .sui-nav {
            position: relative;
            left: 0;
            display: block;
            float: left;
            margin: 0 10px 0 0;

            li {
              float: left;
              line-height: 18px;

              a {
                display: block;
                cursor: pointer;
                padding: 11px 15px;
                color: #777;
                text-decoration: none;
              }

              &.active {
                a {
                  background: #e1251b;
                  color: #fff;
                }
              }
            }
          }
        }
      }

      .goods-list {
        margin: 20px 0;

        ul {
          display: flex;
          flex-wrap: wrap;

          li {
            height: 100%;
            width: 20%;
            margin-top: 10px;
            line-height: 28px;

            .list-wrap {
              .p-img {
                padding-left: 15px;
                width: 215px;
                height: 255px;

                a {
                  color: #666;

                  img {
                    max-width: 100%;
                    height: auto;
                    vertical-align: middle;
                  }
                }
              }

              .price {
                padding-left: 15px;
                font-size: 18px;
                color: #c81623;

                strong {
                  font-weight: 700;

                  i {
                    margin-left: -5px;
                  }
                }
              }

              .attr {
                padding-left: 15px;
                width: 85%;
                overflow: hidden;
                margin-bottom: 8px;
                min-height: 38px;
                cursor: pointer;
                line-height: 1.8;
                display: -webkit-box;
                -webkit-box-orient: vertical;
                -webkit-line-clamp: 2;

                a {
                  color: #333;
                  text-decoration: none;
                }
              }

              .commit {
                padding-left: 15px;
                height: 22px;
                font-size: 13px;
                color: #a7a7a7;

                span {
                  font-weight: 700;
                  color: #646fb0;
                }
              }

              .operate {
                padding: 12px 15px;

                .sui-btn {
                  display: inline-block;
                  padding: 2px 14px;
                  box-sizing: border-box;
                  margin-bottom: 0;
                  font-size: 12px;
                  line-height: 18px;
                  text-align: center;
                  vertical-align: middle;
                  cursor: pointer;
                  border-radius: 0;
                  background-color: transparent;
                  margin-right: 15px;
                }

                .btn-bordered {
                  min-width: 85px;
                  background-color: transparent;
                  border: 1px solid #8c8c8c;
                  color: #8c8c8c;

                  &:hover {
                    border: 1px solid #666;
                    color: #fff !important;
                    background-color: #666;
                    text-decoration: none;
                  }
                }

                .btn-danger {
                  border: 1px solid #e1251b;
                  color: #e1251b;

                  &:hover {
                    border: 1px solid #e1251b;
                    background-color: #e1251b;
                    color: white !important;
                    text-decoration: none;
                  }
                }
              }
            }
          }
        }
      }

      .page {
        width: 733px;
        height: 66px;
        overflow: hidden;
        float: right;

        .sui-pagination {
          margin: 18px 0;

          ul {
            margin-left: 0;
            margin-bottom: 0;
            vertical-align: middle;
            width: 490px;
            float: left;

            li {
              line-height: 18px;
              display: inline-block;

              a {
                position: relative;
                float: left;
                line-height: 18px;
                text-decoration: none;
                background-color: #fff;
                border: 1px solid #e0e9ee;
                margin-left: -1px;
                font-size: 14px;
                padding: 9px 18px;
                color: #333;
              }

              &.active {
                a {
                  background-color: #fff;
                  color: #e1251b;
                  border-color: #fff;
                  cursor: default;
                }
              }

              &.prev {
                a {
                  background-color: #fafafa;
                }
              }

              &.disabled {
                a {
                  color: #999;
                  cursor: default;
                }
              }

              &.dotted {
                span {
                  margin-left: -1px;
                  position: relative;
                  float: left;
                  line-height: 18px;
                  text-decoration: none;
                  background-color: #fff;
                  font-size: 14px;
                  border: 0;
                  padding: 9px 18px;
                  color: #333;
                }
              }

              &.next {
                a {
                  background-color: #fafafa;
                }
              }
            }
          }

          div {
            color: #333;
            font-size: 14px;
            float: right;
            width: 241px;
          }
        }
      }
    }
  }
}
</style>
