<template>
    <div class="container">
        <list class="list">
            <cell>
                <nav-bar @historyShow="historyShow"></nav-bar>
                <recommend-list :recommendList="recommendList" @SidebarShow="SidebarShow" v-if="!errorButton"></recommend-list>
            </cell>
            <cell v-for="(brandListInfo,index) in brandList" :ref="indexNav[index]">
                <brand-list :brandListInfo="brandListInfo" :index="indexNav[index]" @SidebarShow="SidebarShow"></brand-list>
            </cell>
            <cell>
                <other-brandList v-if="moreBrandShow" :otherBrandList="otherBrandList" @SidebarShow="SidebarShow"></other-brandList>
            </cell>
        </list>

        <!--nav导航-->
        <index-nav :indexNav="indexNav" @anchor="anchor"></index-nav>

        <!--sidebar-->
        <div :class="['sidebar',showSidebar?'sidebar-visible':'']">
            <!--遮罩层-->
            <div v-if="shadeShow" class="shade" @click="sidebarHide"></div>

            <!--内容-->
            <list class="sidebar-content" style="flex: 1" ref="side">
                <cell class="sidebar-header" v-if="showSidebar">
                    <div class="back" @click="sidebarHide">
                        <image src="https://s.kcimg.cn/wap/images/app_icon/back.png" style="width:40px;height:40px"></image>
                        <!--<text class="back-text" :style="{fontFamily:'detail'}">&#x56de;</text>-->
                    </div>
                    <text class="sidebar-header-text">{{sidebarList.brandName}}</text>
                </cell>
                <cell v-for="(data,index) in sidebarList.seriesList" class="sidebar-module" v-if="showSidebar">
                    <div class="title" v-if="notHistoryTitle">
                        <text class="title-text">{{data[0].info.F_SubCategoryName}}</text>
                    </div>
                    <div v-for="(ele,number) in data" class="sidebar-detail">
                        <div class="truck-info" :href="'https://product.m.360che.com' + ele.series_url.detailUrl" @click="goInfo(ele)">
                            <image v-if="ele.img" style="width:150px;height:100px;" :src="ele.img[0].src"></image>
                            <div v-else class="not-image">
                                <image src="https://s.kcimg.cn/wap/images/app_icon/logo_2.png" style="width:131px;height:48px"></image>
                                <!--<text :style="{fontFamily:'detail',fontSize:'40px',color:'#999'}">&#x5361;</text>-->
                            </div>
                            <div class="truck-detail">
                                <text class="truck-name">{{ele.info.F_ShortName?ele.info.F_ShortName:ele.info.F_SeriesName + ele.info.F_SubCategoryName}}</text>
                                <text class="truck-price">{{ele.priceScope.min ? (ele.priceScope.min == ele.priceScope.max ? ele.priceScope.min + '万元' : ele.priceScope.min + '~' + ele.priceScope.max + '万元') : '暂无报价'}}</text>
                            </div>
                        </div>
                    </div>
                </cell>
                <!--空历史记录-->
                <cell v-if="emptyHistory" class="empty-history">
                    <image src="https://s.kcimg.cn/wap/images/app_icon/bad.png" style="width:155px;height:100px;"></image>
                    <!--<text :style="{fontFamily:'detail',fontSize:'100px',color:'#a1c6f5'}">&#x65e0;</text>-->
                    <text class="empty-history-text">暂无浏览记录，赶紧去看吧~</text>
                </cell>
                <!--弹层内容-->
                <cell v-if="showSidebar && seriesMoreShow" v-for="(data,index) in sidebarList.otherSeriesList">
                    <div class="title">
                        <text class="title-text">{{data[0].info.F_SubCategoryName}}</text>
                    </div>
                    <div v-for="(ele,number) in data" class="sidebar-detail">
                        <div class="truck-info" :href="'https://product.m.360che.com' + ele.series_url.detailUrl" @click="goInfo(ele)">
                            <image v-if="ele.img" style="width:150px;height:100px;" :src="ele.img[0].src"></image>
                            <div v-else class="not-image">
                                <image src="https://s.kcimg.cn/wap/images/detail/productApp/truck-home.png" style="width:131px;height:48px;"></image>
                                <!--<text :style="{fontFamily:'detail',fontSize:'40px',color:'#999'}">&#x5361;</text>-->
                            </div>
                            <div class="truck-detail">
                                <text class="truck-name">{{ele.info.F_ShortName?ele.info.F_ShortName:ele.info.F_SeriesName + ele.info.F_SubCategoryName}}</text>
                                <text class="truck-price">{{ele.priceScope.min ? (ele.priceScope.min == ele.priceScope.max ? ele.priceScope.min + '万元' : ele.priceScope.min + '~' + ele.priceScope.max + '万元') : '暂无报价'}}</text>
                            </div>
                        </div>
                    </div>
                </cell>
                <cell v-if="showSidebar && moreSeriesButtonShow" class="seriesMore">
                    <div class="seriesMore-button" @click="seriesMore">
                        <text class="seriesMore-button-text">{{seriesMoreShow?'收起':'展开'}}更多车系</text>
                        <image ref="more" src="https://s.kcimg.cn/wap/images/app_icon/down.png" style="width:44px;height:30px;"></image>
                        <!--<text  ref="more" :style="{fontFamily:'detail',color:'#999',fontSize:'32px'}">&#x4e0b;</text>-->
                    </div>
                </cell>
            </list>

            <!--清除历史记录-->
            <div class="clear-history" ref="clearHostory" @click="clearHistory">
                <!--当点开历史记录 && 列表不为空的时候在显示-->
                <div v-if="clearHistoryShow && !emptyHistory" class="clear-history-button">
                    <image src="https://s.kcimg.cn/wap/images/app_icon/delete.png" style="width:32px;height:32px;margin-right:10px"></image>
                    <!--<text :style="{fontFamily:'detail',fontSize:'32px',color:'#333',marginRight:'10px'}">&#x5220;</text>-->
                    <text class="clear-text">清除全部浏览记录</text>
                </div>
            </div>
        </div>

        <!--加载失败重新加载按钮-->
        <div class="error-button" v-if="errorButton">
            <image src="https://s.kcimg.cn/wap/images/detail/productApp/not_network.png" style="width:400px;height:300px;"></image>
            <text class="not_network">当前网络不可用</text>
            <div class="reload" @click="getBrandsData">
                <text class="reload-text">点击刷新</text>
            </div>
        </div>

        <!--网络失败弹窗-->
        <error-pop v-if="errorPopShow" :errorPopText="errorPopText"></error-pop>
    </div>
</template>

<script type="text/babel">
    import navBar from '../components/brandList/navBar.vue'
    import recommendList from '../components/brandList/recommendBrand.vue'
    import brandList from '../components/brandList/brandList.vue'
    import indexNav from '../components/brandList/indexNav.vue'
    import otherBrandList from '../components/brandList/otherBrandList.vue'
    import errorPop from '../components/errorPop.vue'
    import mixins from '../mixins'


    let stream = weex.requireModule('stream')
    let dom = weex.requireModule('dom')
    let animation = weex.requireModule('animation')
    let modal = weex.requireModule('modal')
    let storage = weex.requireModule('storage')
    let thaw = weex.requireModule('THAW')
    var globalEvent = weex.requireModule('globalEvent');
//    let navigator = weex.requireModule('navigator')

    export default {
        data(){
            return {
                //推荐品牌列表
                recommendList: [],
                //品牌列表
                brandList: [],
                //品牌导航
                indexNav:[],
                //更多品牌列表
                otherBrandList:[],
                //展开更多品牌按钮隐藏
                moreBrandShow:false,
                //sidebar内容
                sidebarList: {},
                //sidebar内容标题
                notHistoryTitle:true,
                //sidebar显示与隐藏
                showSidebar: false,
                //sidebar遮罩层显示和隐藏
                shadeShow: false,
                //展开更多车系,
                seriesMoreShow:false,
                //更多车系按钮
                moreSeriesButtonShow:true,
                //是否显示清除历史记录
                clearHistoryShow:false,
                //历史记录是否为空
                emptyHistory:false,
                //网络出错弹层
                errorPopShow:false,
                //提示文案
                errorPopText:'当前网络不可用，请检查网络设置',
                //网络出错重新加载按钮
                errorButton:false,
            }
        },
        methods: {
            alert(name){
                modal.alert({
                    message: name,
                    duration: 0.3
                })
            },
            //发送请求
            getData(repo, callback){
                return stream.fetch({
                    method: 'GET',
                    type: 'json',
                    url: repo
                }, callback)
            },

            //点击锚点跳转
            anchor(index){
                dom.scrollToElement(this.$refs[index][0], {offset: 0})
            },
            //sidebar内容点击跳转
            goInfo(ele){
                let p4 = ele.info.F_BrandId + '_' + ele.info.F_CateId + '_' + ele.info.F_SubCategoryId;
                let p5 = ele.info.F_SeriesId;
                //存储p4
                storage.setItem('p4',p4)
                storage.setItem('p5',p5)


                //存储历史记录
                storage.getItem('historyData',data => {
                    if(data.result == 'success'){
                        let list = JSON.parse(data.data);
                        let hasHistory = true;
                        list.forEach(function(res,index){
                            res.forEach(function(el,i){
                                //历史记录已经存在
                                if(el.imgCount == ele.imgCount){
                                    hasHistory = false;
                                }
                            })
                        });
                        //历史记录没有包含
                        if(hasHistory){
                            list[0].push(ele);
                            storage.setItem('historyData',JSON.stringify(list),success => {
                                if(success.result == 'success'){
                                    //成功之后跳转url
//                                    thaw.goUrl('https://product.m.360che.com' + ele.series_url.detailUrl);
                                    this.showLoading();
                                    //存储车系数据
                                    storage.setItem('seriesInfo',JSON.stringify(ele.info),() => {
                                        //跳转
                                        this.goWeexUrl('series.weex.js');
                                    })

                                }
                            })
                        }else{
                            //已有的历史记录直接跳转
//                            thaw.goUrl('https://product.m.360che.com' + ele.series_url.detailUrl)
                            this.showLoading()
                            //存储车系数据
                            storage.setItem('seriesInfo',JSON.stringify(ele.info),() => {
                                //跳转
                                this.goWeexUrl('series.weex.js');
                            })


                        }
                    }else{
                        let historyData = [];
                        historyData[0] = [];
                        historyData[0].push(ele);
                        storage.setItem('historyData',JSON.stringify(historyData),success => {
                            if(success.result == 'success'){
                                //成功之后跳转url
//                                thaw.goUrl('https://product.m.360che.com' + ele.series_url.detailUrl)
                                this.showLoading();
                                //存储车系数据
                                storage.setItem('seriesInfo',JSON.stringify(ele.info),() => {
                                    //跳转
                                    this.goWeexUrl('series.weex.js');
                                })
                            }
                        })
                    }
                });

            },
            //显示历史记录
            historyShow(){
                //隐藏展开更多车系按钮
                this.moreSeriesButtonShow = false;
                //隐藏sidebar内容标题
                this.notHistoryTitle = false;
                this.sidebarList.brandName = '浏览记录';
                storage.getItem('historyData',ele => {
                    if(ele.result == 'success'){
                        this.sidebarList.seriesList = JSON.parse(ele.data);
                    }else{
                        this.sidebarList.seriesList = [];
                        //当内容为空的时候显示历史记录为空
                        this.emptyHistory = true;
                    }

                    //是否显示清除历史记录
                    this.clearHistoryShow = true;
                    this.showSidebar = true;
                    this.shadeShow = true;
                    this.$nextTick(function(){
                        var side = this.$refs.side;
                        animation.transition(side, {
                            styles: {
                                transform: 'translate(0,0)'
                            },
                            duration: 300, //ms
                            timingFunction: 'linear',
                            delay: 0 //ms
                        });
                        //展开清除历史记录
                        var clearHostory = this.$refs.clearHostory;
                        animation.transition(clearHostory, {
                            styles: {
                                transform: 'translate(0,0)'
                            },
                            duration: 300, //ms
                            timingFunction: 'linear',
                            delay: 0 //ms
                        })
                    })
                })
            },
            //点击清除历史记录
            clearHistory(){
                storage.getItem('historyData',ele => {
                    if(ele.result == 'success'){
                        this.sidebarList.seriesList = [];
                        this.emptyHistory = true;
                        storage.removeItem('historyData')
                    }
                })
            },
            //显示sidebar
            SidebarShow(ele,other){
                console.log(ele)
//                let ajaxUrl = ele.url.replace('product.m.360che.com', 'product-yufabu.m.360che.com');
//                ajaxUrl = ajaxUrl.replace('https', 'http');
                let ajaxUrl = ele.url + '&isJson=1&noIndex=1';

//                let ajaxUrl = url + '&isJson=1&noIndex=1';

                //存储sidebar内容 ：怕数据产生变化，暂时不存储
//                storage.getItem('SidebarData',(ele) => {
//                    if(ele.result == 'success'){
//                        let data = JSON.parse(ele.data);
//                        if(ele.data[ajaxUrl]){
//
//                        }
//                    }
//                });

                this.getData(ajaxUrl, data => {

                    let p3 = ele.id + '__';
                    if(!ele.id){
                        p3 = ele.F_BrandId + '__'
                    }

                    if (data.ok) {
                        //大数据发送事件类
                        this.collect({
                            'p3':p3
                        })

                        //如果没有更多车系,隐藏展开更多车系按钮
                        if(data.data.otherSeriesList.length == 0){
                            this.moreSeriesButtonShow = false;
                        }

                        this.sidebarList = data.data;

                        //如果点击更多品牌，隐藏车系 && 直接显示sidebar的更多车系
                        if(other == 'other'){
                            this.seriesMoreShow = true;
                            this.moreSeriesButtonShow = false;
                        }

                        this.showSidebar = true;
                        this.shadeShow = true;
                        this.$nextTick(() => {
                            var side = this.$refs.side;
                            animation.transition(side, {
                                styles: {
                                    transform: 'translate(0,0)'
                                },
                                duration: 300, //ms
                                timingFunction: 'linear',
                                delay: 0 //ms
                            })
                        })
                    } else {
                        this.errorPopShow = true;
                    }
                });


            },
            //隐藏sidebar
            sidebarHide(){
                //隐藏遮罩层
                this.shadeShow = false;
                //隐藏更多车系按钮
                this.moreSeriesButtonShow = true;
                //显示sidebar内容标题
                this.notHistoryTitle = true;

                //如果打开的是历史记录 && 隐藏清除历史记录
                if(this.clearHistoryShow){
                    //展开清除历史记录
                    var clearHostory = this.$refs.clearHostory;
                    animation.transition(clearHostory, {
                        styles: {
                            transform: 'translate(750px,0)'
                        },
                        duration: 300, //ms
                        timingFunction: 'linear',
                        delay: 0 //ms
                    },ele => {
                        this.clearHistoryShow = false;
                        //隐藏历史记录为空icon
                        this.emptyHistory = false;
                    });
                }

                var side = this.$refs.side;
                animation.transition(side, {
                    styles: {
                        transform: 'translate(750px,0)'
                    },
                    duration: 300, //ms
                    timingFunction: 'linear',
                    delay: 0 //ms
                }, () => {
                    //隐藏sidebar
                    this.showSidebar = false;
                    //隐藏更多车系
                    this.seriesMoreShow = false;
                })

            },
            //展开更多车系
            seriesMore(){
                this.seriesMoreShow = !this.seriesMoreShow;
                var more = this.$refs.more;
                let rotate = 180;
                if(this.seriesMoreShow){
                    rotate = 180
                }else{
                    rotate = 0
                }
                animation.transition(more, {
                    styles: {
                        transform: 'rotate(' + rotate + 'deg)'
                    },
                    duration: 300, //ms
                    timingFunction: 'ease-out',
                    delay: 0 //ms
                })
            },
            //点击重新加载数据
            getBrandsData(){
                this.getData(this.ajaxUrl() +'/index.php?r=api/gethotbrandlist&haveGroup=1&noIndex=1', data => {
//                this.alert(this.ajaxUrl() + '/index.php?r=api/gethotbrandlist&haveGroup=1&noIndex=1')
//                this.alert(JSON.stringify(data))
                    if (data.ok) {
                        //推荐品牌列表
                        this.recommendList = data.data.recommend;
                        //品牌列表
                        this.brandList = data.data.brandList;
                        //nav导航
                        this.indexNav = data.data.letter;
                        //更多品牌列表
                        this.otherBrandList = data.data.otherBrandList;

                        //显示展开更多品牌按钮
                        this.$nextTick(function(){
                            this.moreBrandShow = true;
                        });

                        this.errorPopShow = false;
                        this.errorButton = false;

//                        storage.setItem('brandsData', JSON.stringify(data));
                    } else {
                        if(!this.moreBrandShow){
                            this.errorPopShow = true;
                            this.errorButton = true;
                        }
                    }
                });
            }
        },
        created(){
            //存储deviceId
            if(weex.config.deviceId){
                storage.setItem('deviceId',weex.config.deviceId)
            }

            //存储uid
            if(weex.config.uid){
                storage.setItem('uid',weex.config.uid)
            }

//            //发送GA统计
            this.collect({
            })

            //iconFont字体
            dom.addRule('fontFace',{
                'fontFamily':'detail',
                'src':"url(\'https://at.alicdn.com/t/font_1z3q14vor7h3q5mi.ttf\')"
            });

            //请求接口,查看版本号
            this.getData(this.ajaxUrl() + '/index.php?r=weex/index/version',ele => {
                if(ele.ok && ele.data.info == 'ok'){
                    storage.setItem('versions',ele.data.version)
                }
            })

//            storage.getItem('brandsData', ele => {
//                if (ele.result == 'success') {
//                    let data = JSON.parse(ele.data);
//                    this.recommendList = data.data.recommend;
//                    this.brandList = data.data.brandList;
////                    显示展开更多品牌按钮
//                    this.moreBrandShow = true;
//                } else {
//            this.alert(1)

            //获取品牌缓存

            storage.getItem('brandsData',ele => {
                if(ele.result == 'success'){
                    let data = JSON.parse(ele.data)
                    //推荐品牌列表
                    this.recommendList = data.data.recommend;
                    //品牌列表
                    this.brandList = data.data.brandList;
                    //nav导航
                    this.indexNav = data.data.letter;
                    //更多品牌列表
                    this.otherBrandList = data.data.otherBrandList;

                    //显示展开更多品牌按钮
                    this.$nextTick(function(){
                        this.moreBrandShow = true;
                    });
                }

                //请求品牌数据
                this.getBrandsData()
            })
        },
        components: {
            navBar,
            recommendList,
            brandList,
            indexNav,
            otherBrandList,
            errorPop
        },
        watch:{
            errorPopShow(){
                if(this.errorPopShow){
                    this.Time = setTimeout(() => {
                        this.errorPopShow = false;
                        clearTimeout(this.Time)
                    },1000)
                }
            }
        },
        mounted(){
//            navigator.push({
//                url: 'https://192.168.1.120:8888/dist/series.weex.js',
//                animated: "true"
//            })
            globalEvent && globalEvent.addEventListener("onSendLocation",(e) => {
                if(e.state == 'success') {
                    storage.setItem('getlocationInfo', JSON.stringify(e))
                }
            })

            //调取地理位置信息
            thaw && thaw.getLocation();

        }
    }
</script>

<style scoped>
    .container {
        flex: 1;
        /*overflow: hidden;*/
    }

    .list {
        flex: 1;
    }

    .sidebar {
        position: fixed;
        top: 0;
        right: 0;
        bottom: 0;
        left: 750px;
        padding-left: 140px;
        align-items: flex-end;
    }

    .sidebar-visible {
        left: 0;
    }

    .shade {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        bottom: 0;
        background-color: rgba(0, 0, 0, .6);
        /*transform: translate(750px, 0);*/
    }

    /*.shade-visible {*/
        /*transform: translate(0, 0);*/
    /*}*/

    .sidebar-content {
        flex: 1;
        width:600px;
        background-color: #f5f5f5;
        transform: translate(750px, 0);
    }

    .sidebar-header {
        position: relative;
        padding-left: 88px;
        padding-right: 88px;
        height: 90px;
        align-items: center;
        justify-content: center;
        border-bottom-width:1px;
        border-bottom-style: solid;
        border-bottom-color:#eee;
        background-color: #fff;
    }

    .sidebar-header-text {
        color: #333;
        font-size: 36px;
    }

    .back {
        position: absolute;
        left: 0;
        top: 0;
        width: 88px;
        height: 90px;
        align-items: center;
        justify-content: center;
    }
    .title{
        padding-left:20px;
        height: 48px;
        background-color:#f5f5f5;
        justify-content: center;
    }
    .title-text{
        color:#666;
        font-size:28px;
    }
    .sidebar-detail{
        padding-left:20px;
        padding-right:20px;
        background-color: #fff;
    }
    .truck-info{
        display: flex;
        padding-top:20px;
        padding-bottom:20px;
        flex-direction: row;
        align-items: center;
        justify-content: center;
        border-bottom-width:1px;
        border-bottom-style: solid;
        border-bottom-color:#eee;
    }
    .truck-info:active{
        background-color:rgba(0,0,0,.2);
    }
    .truck-detail{
        flex:1;
        padding-left:20px;
    }
    .not-image{
        width:150px;
        height:100px;
        background-color:#efefef;
        justify-content: center;
        align-items: center;
    }
    .truck-name{
        color:#333;
        font-size:32px;
    }
    .truck-price{
        color:#f60;
        font-size:24px;
        padding-top: 20px;
    }
    .seriesMore{
        padding-top:40px;
        padding-right:40px;
        padding-bottom:40px;
        padding-left:40px;
    }
    .seriesMore-button{
        height:88px;
        background-color:#fff;
        flex-direction: row;
        justify-content: center;
        align-items: center;
    }
    .seriesMore-button-text{
        color: #999;
        font-size: 32px;
        margin-right:20px;
    }
    .unfold{
        transform:rotate(90deg)
    }
    .clear-history{
        /*position:fixed;*/
        transform: translate(750px,0);
    }
    .clear-history-button{
        width:600px;
        height:88px;
        background-color:#fff;
        flex-direction: row;
        justify-content: center;
        align-items:center;
    }
    .clear-text{
        color:#333;
        font-size:32px;
    }
    .empty-history{
        padding-top:50px;
        justify-content: center;
        align-items: center;
    }
    .empty-history-text{
        color:#999;
        font-size:32px;
        padding-top:20px;
    }
    .error-button{
        position:absolute;
        left: 0 ;
        right:0;
        top:0;
        bottom:0;
        align-items: center;
        justify-content: center;
    }
    .reload{
        width:180px;
        height:64px;
        justify-content: center;
        align-items: center;
        margin-top: 20px;
        border-top-left-radius:10px;
        border-top-right-radius:10px;
        border-bottom-left-radius:10px;
        border-bottom-right-radius:10px;
        background-color:#1571E5;
    }
    .not_network{
        margin-top:10px;
        color:#999;
    }
    .reload-text{
        color:#fff;
        font-size:28px;
    }
</style>
