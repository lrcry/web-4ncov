<template>
    <div>
        <Nav/>
        <dataMap v-if="featureToggle" :title="title" :dataTime="dataTime"/>
        <section class="listing">
            <div class="gap"></div>
            <p class="listing__title">查询物资寻求方</p>
            <el-tabs @tab-click="handleTabSwitch">
                <el-tab-pane v-for="cat in categories" :key="cat" v-bind:label="cat"></el-tab-pane>
            </el-tabs>
            <MaterialList :materials="materials" quantityTitle="需" addressTitle="收货地址"></MaterialList>
            <el-button class="materials-loadmore" v-on:click="loadMore" v-bind:loading="loadingMore"
                       v-bind:disabled="!hasNextPage">
                {{ hasNextPage ? '加载更多' : '没有更多' }}
            </el-button>
        </section>
        <div v-if="isHospital || !isLogin" class="publish-button">
            <a href="/required-materials">发布寻求</a>
        </div>
    </div>
</template>

<script>
    import mapRefreshMixin from '../utils/mapRefreshMixin'
    import Nav from '../components/Nav'
    import dataMap from '../components/dataMap'
    import MaterialList from '../components/MaterialList'
    import categories from '../utils/MaterialCategories'
    import RequiredMaterialService from '../services/RequiredMaterial'
    import UserService from '../services/user'
    import featureToggle from '../utils/FeatureToggle'

    export default {
        name: 'required-materials-overview',
        components: { Nav, dataMap, MaterialList },
        mixins: [mapRefreshMixin],
        data() {
            return {
                title: '医疗物资需求分布地图',
                dataTime: '2020.01.29 15:30',
                materials: [],
                categories,
                page: 1,
                size: 10,
                loadingMore: false,
                hasNextPage: true,
                selectedCategory: categories[0],
                isHospital: UserService.isHospital(),
                isLogin: UserService.isLogin(),
                featureToggle
            }
        },
        created() {
            RequiredMaterialService.getRequiredMaterials(this.page, this.size, this.selectedCategory).then(materials => {
                if (materials.length === 0) {
                    this.hasNextPage = false
                    return
                }
                this.materials = materials
            })
        },
        computed: {},
        methods: {
            dataProcessor() {
                this.option.visualMap = {
                    left: 'right',
                    min: 0,
                    max: 1000,
                    inRange: {
                        color: ['#fff', '#ff3908']
                    },
                    text: ['缺口', '富余'], // 文本，默认为数值文本
                    calculable: true,
                    itemWidth: 10,
                    itemHeight: 100
                }
                this.option.series = [
                    {
                        name: 'china map',
                        type: 'map',
                        roam: false,
                        map: 'china',
                        emphasis: {
                            label: {
                                show: true
                            }
                        },
                        // 文本位置修正
                        textFixed: {
                            Alaska: [20, -20]
                        },
                        data: [
                            { name: '北京', value: 500 },
                            { name: '山东', value: 10 },
                            { name: '湖北', value: 1200 },
                            { name: '上海', value: 1000 }
                        ]
                    }
                ]
                return this.option
            },
            handleTabSwitch(tab) {
                this.materials = []
                this.selectedCategory = tab.label
                this.page = 1
                this.loadingMore = false
                this.hasNextPage = true
                RequiredMaterialService.getRequiredMaterials(this.page, this.size, this.selectedCategory).then(materials => {
                    if (materials.length === 0) {
                        this.hasNextPage = false
                        return
                    }
                    this.materials = materials
                })
            },
            loadMore() {
                if (!this.hasNextPage) {
                    return
                }
                this.loadingMore = true
                ++this.page
                RequiredMaterialService.getRequiredMaterials(this.page, this.size, this.selectedCategory).then(materials => {
                    this.loadingMore = false
                    if (materials.length === 0) {
                        this.hasNextPage = false
                        return
                    }
                    this.materials = this.materials.concat(materials)
                })
            }
        }
    }
</script>

<style scoped>
    .listing {
        padding: 0 0.5rem;
        height: 10rem;
    }

    .gap {
        margin-top: 0.2rem;
        height: 0.2rem;
        background-color: rgba(242, 242, 242, 1);
    }

    .listing__title {
        border-left: 0.08rem solid #09f;
        font-size: 0.35rem;
        padding-left: 0.08rem;
        margin-top: 0.2rem;
    }

    .materials-loadmore {
        width: 100%;
    }

    .publish-button {
        height: 1.1rem;
        width: 1.1rem;
        padding: 0.1rem;
        display: inline-block;
        position: fixed;
        bottom: 1rem;
        right: 0.2rem;
        border-radius: 50%;
        border: 1px solid #fff;
        text-align: center;
        background-color: #f00;
        z-index: 9999;
    }

    .publish-button a {
        height: 0.8rem;
        width: 0.8rem;
        display: inline-block;
        text-decoration: none !important;
        line-height: 0.5rem;
        color: #fff !important;
    }
</style>
