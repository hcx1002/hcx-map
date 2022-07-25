<template>
  <div>
    <div class="map">
      <div id="container"></div>
      <div class="panel">
        <el-autocomplete
          popper-class="my-autocomplete"
          v-model="keyword"
          :fetch-suggestions="querySearch"
          placeholder="请输入地址"
          @select="handleSelect"
          style="width: 300px"
		  suffix-icon="none">
		<i class="el-icon-search" slot="suffix" style="cursor: pointer;color: #000" @click="searchByKeyword()"></i>
          <template slot-scope="{ item }">
            <div>
              <div>{{ item.title }}</div>
              <div style="color: #aaa;font-size: 13px">{{ item.address }}</div>
            </div>
          </template>
        </el-autocomplete>
      </div>
    </div>
  </div>
</template>

<script>
/**
 * map  地图地址坐标选择器
 * @author HCX
 * @property {Number}	 latitude			地图中心经度
 * @property {Number}	 longitude			地图中心纬度
 * @event    {Function}  addressInfo        点击地图时触发
 * @event    {Function}  handleSelect       选择搜索列表触发
 * @example  <my-map @addressInfo="addressInfo" @handleSelect="handleSelect"></my-map>
 */
import elAutocomplete from 'element-ui'
export default {
  name: 'map',
  props:{
	latitude:{
		type:Number,
		default:39.984104
	},
	longitude:{
		type:Number,
		default:116.307503
	}
  },
  data(){
    return{
      suggest: '',
      markers: '',
      infoWindowList: '',
      suggestionList: [],
      search: '',
      keyword: '',
      map: '',
      geocoder:'',
      addressInfo:{}
    }
  },
  created() {
    // this.loadScript()

  },
  mounted() {
    this.initMap()
  },
  methods:{
    initMap(){
      //初始化地图
      let that = this
      var center = new TMap.LatLng(this.latitude, this.longitude)//设置中心点坐标
      this.map = new TMap.Map('container', {
        center: center
      })
      this.search = new TMap.service.Search({ pageSize: 10 }) // 新建一个地点搜索类
      this.geocoder = new TMap.service.Geocoder();
      this.suggest = new TMap.service.Suggestion({
        // 新建一个关键字输入提示类
        pageSize: 10, // 返回结果每页条目数
        region: '', // 限制城市范围
        regionFix: true // 搜索无结果时是否固定在当前城市
      })
      this.markers = new TMap.MultiMarker({
        map: this.map,
        geometries: []
      })
      this.infoWindowList = Array(10)
      //绑定点击事件
      this.map.on('click', function(evt) {
        var lat = evt.latLng.getLat().toFixed(6)
        var lng = evt.latLng.getLng().toFixed(6)
        let location = new TMap.LatLng(lat, lng);
        that.markers.updateGeometries([
          {
            id: 'main', // 点标注数据数组
            position: location,
          },
        ]);
        // 将给定的坐标位置转换为地址
        that.geocoder.getAddress({ location: location }).then((result) => {
          that.$emit("addressInfo",result)
        });
      })
    },
    getSuggestions() {
      let that = this
      // 使用者在搜索框中输入文字时触发
      if (this.keyword) {
        return new Promise((resolve, reject)=>{
          this.suggest.getSuggestions({ keyword: this.keyword, location: this.map.getCenter() })
            .then((result) => {
              console.log(result)
              that.suggestionList = result.data
              resolve(result.data)
            }).catch((error) => {
            console.log(error)
            reject(error)
          })
        })

      }
    },
    searchByKeyword() {
      let that = this
      // 关键字搜索功能
      this.infoWindowList.forEach((infoWindow) => {
        infoWindow.close()
      })
      this.infoWindowList.length = 0
      this.markers.setGeometries([])
      this.search.searchRectangle({
        keyword: this.keyword,
        bounds: this.map.getBounds()
      }).then((result) => {
        console.log(result)
        result.data.forEach((item, index) => {
          var geometries = that.markers.getGeometries()
          var infoWindow = new TMap.InfoWindow({
            map: that.map,
            position: item.location,
            content: `<h3>${item.title}</h3><p>地址：${item.address}</p>`,
            offset: { x: 0, y: -50 }
          })
          infoWindow.close()
          that.infoWindowList[index] = infoWindow
          geometries.push({
            id: String(index),
            position: item.location
          })
          that.markers.updateGeometries(geometries)
          that.markers.on('click', (e) => {
            that.infoWindowList[Number(e.geometry.id)].open()
          })
        })
      })
    },
    handleSelect(res){
      this.$emit("handleSelect",res)
      this.infoWindowList.forEach((infoWindow) => {
        infoWindow.close()
      })
      this.infoWindowList.length = 0
      this.keyword = res.title
      this.markers.setGeometries([])
      this.markers.updateGeometries([
        {
          id: '0', // 点标注数据数组
          position: res.location
        }
      ])
      let infoWindow = new TMap.InfoWindow({
        map: this.map,
        position: res.location,
        content: `<h3>${res.title}</h3><p>地址：${res.address}</p>`,
        offset: { x: 0, y: -50 }
      })
      this.infoWindowList.push(infoWindow)
      this.map.setCenter(res.location)
      this.markers.on('click', (e) => {
        this.infoWindowList[Number(e.geometry.id)].open()
      })
    },
    querySearch(queryString, cb) {
      console.log(queryString)
      // 调用 callback 返回建议列表的数据
      if (queryString){
        this.getSuggestions().then(res=>{
          cb(res);
        })
      }
    }
  }
}
</script>

<style scoped>
#container{
  z-index: 1;
}
.map{
  position: relative
}
.panel{
  position: absolute;
  top: 20px;
  left: 20px;
  z-index: 2;
}
</style>
