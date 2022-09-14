<script setup>
  import {
    ref,
    reactive,
    computed
  } from 'vue'
  import {onLoad} from '@dcloudio/uni-app'
  const timer = ref(null)
  const kw = ref('')
  const searchResults = ref([])
  const historyList = ref ([])

onLoad(()=>{
  historyList.value = JSON.parse(uni.getStorageSync('kw') || '["a", "app", "apple"]')
}) 
  const input = (e) => {
    if (timer.value) clearTimeout(timer.value)
    timer.value = setTimeout(() => {
      kw.value = e
      getSearchList()

    }, 500)
  }

  const getSearchList = async () => {
    if (kw.value === '') {
      searchResults.value = []
      return
    }
      const { data: res } = await uni.$http.get('/api/public/v1/goods/qsearch', { query: kw.value })
      searchResults.value = res.message
      saveSearchHistory()
  }
  
  
  const gotoDetail=(goods_id)=> {
    uni.navigateTo({
      // 指定详情页面的 URL 地址，并传递 goods_id 参数
      url: '/subpkg/good_detail/good_detail?goods_id=' + goods_id
    })
  }
 const  saveSearchHistory =()=> {
    // 2.1 直接把搜索关键词 push 到 historyList 数组中
      // 1. 将 Array 数组转化为 Set 对象
      const set = new Set(historyList.value)
      // 2. 调用 Set 对象的 delete 方法，移除对应的元素
      set.delete(kw.value)
      // 3. 调用 Set 对象的 add 方法，向 Set 中添加元素
      set.add(kw.value)
      // 4. 将 Set 对象转化为 Array 数组
      historyList.value = Array.from(set)
      uni.setStorageSync('kw', JSON.stringify(historyList.value))
  }
  const SearchHistory = computed(()=>{
    return [...historyList.value].reverse()
  })
  
  const cleanHistory = ()=>{
      // 清空 data 中保存的搜索历史
      historyList.value = []
      // 清空本地存储中记录的搜索历史
      uni.removeStorageSync('kw')
  }
  const gotoGoodsList = (kw)=>{

      uni.navigateTo({
        url: '/subpkg/goods_list/goods_list?query=' + kw
      })
  }
</script>

<template>
  <view class="search-box">
    <!-- 使用 uni-ui 提供的搜索组件 -->
    <uni-search-bar @input="input" :radius="100" cancelButton="none" clearButton="none"></uni-search-bar>
  </view>
  
  <!-- 搜索建议列表 -->
  <view class="sugg-list" v-if="searchResults.length!==0">
    <view class="sugg-item" v-for="(item, i) in searchResults" :key="i" @click="gotoDetail(item.goods_id)">
      <view class="goods-name">{{item.goods_name}}</view>
      <uni-icons type="arrowright" size="16"></uni-icons>
    </view>
  </view>
  
  <view class="history-box" v-else>
    <!-- 标题区域 -->
    <view class="history-title">
      <text>搜索历史</text>
      <uni-icons type="trash" size="17" @click="cleanHistory"></uni-icons>
    </view>
    <!-- 列表区域 -->
    <view class="history-list">
      <uni-tag :text="item" v-for="(item, i) in SearchHistory" :key="i" inverted @click="gotoGoodsList(item)"></uni-tag>
    </view>
  </view>
  

</template>



<style lang="scss">
  .uni-searchbar {
    /* #ifndef APP-NVUE */
    display: flex;
    /* #endif */
    flex-direction: row;
    position: relative;
    padding: 16rpx;
    /* 将默认的 #FFFFFF 改为 #C00000 */
    background-color: #c00000;
  }

  .search-box {
    position: sticky;
    top: 0;
    z-index: 999;
  }
  
  .sugg-list {
    padding: 0 5px;
  
    .sugg-item {
      font-size: 12px;
      padding: 13px 0;
      border-bottom: 1px solid #efefef;
      display: flex;
      align-items: center;
      justify-content: space-between;
  
      .goods-name {
        // 文字不允许换行（单行文本）
        white-space: nowrap;
        // 溢出部分隐藏
        overflow: hidden;
        // 文本溢出后，使用 ... 代替
        text-overflow: ellipsis;
        margin-right: 3px;
      }
    }
  }
  
  .history-box {
    padding: 0 5px;
  
    .history-title {
      display: flex;
      justify-content: space-between;
      align-items: center;
      height: 40px;
      font-size: 13px;
      border-bottom: 1px solid #efefef;
    }
  
    .history-list {
      display: flex;
      flex-wrap: wrap;
  
      .uni-tag {
        margin-top: 5px;
        margin-right: 5px;
        display: block;
      }
    }
  }
</style>
