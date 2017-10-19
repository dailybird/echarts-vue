<style scoped>
    .main-container {
        position: relative;
    }

    .wrap-container,
    .loading,
    .shadow {
        position: absolute;
    }

    .wrap-container,
    .echarts {
        width: 100%;
        height: 100%;
    }

    .shadow,
    .loading {
        width: 100%;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: .9rem;
        color: #8590a6;
    }
</style>

<template>
    <div class="main-container" ref="selfEcharts">
        <div class="loading" v-show="isLoading">
            数据加载中...
        </div>
        <div class="shadow" v-show="!isLoading && isOptionAbnormal">
            数据为空
        </div>
        <div class="wrap-container" v-show="!isLoading && !isOptionAbnormal">
            <div class="echarts" :id="randomId"></div>
        </div>
    </div>
</template>

<script>
    import echarts from 'echarts'
    import _ from 'lodash'

    export default {
        name: 'echarts',
        props: {
            option: {
                default () {
                    return {}
                }
            },

            /**
             * 加载提示信息显示标识
             */
            isLoading: {
                type: Boolean,
                default: false
            },

            /**
             * resize 调整标识
             */
            resizeSignature: {
                default: ''
            },
            /**
             * 需要注册的地图
             */
            maps: {
                type: Array,
                default () {
                    return []
                }
            }
        },
        data () {
            return {
                /**
                 * 为该组件生成具有唯一 ID DOM
                 */
                randomId: 'echarts-dom' + Date.now() + Math.random(),
                myEcharts: null,
                scrollThrottle: 200,
                windowResizeThrottle: 500,
                /**
                 * option 是否不合法
                 */
                isOptionAbnormal: false,

                /**
                 * 滚动事件实名函数
                 */
                scrollEvent: _.throttle(this.checkPosition, this.scrollThrottle),

                /**
                 * 页面是否已经滚动到了合适位置
                 */
                isPositionReady: false
            }
        },
        created () {
            /**
             * 如果使用方需要调用地图组件
             * 即注册地图元素
             * 可通过传参的方式传入
             */
            let maps = this.maps
            if (maps.count !== 0) {
                for (let index in maps) {
                    let map = maps[index]
                    if (isObject(map) && map['name'] !== undefined && map['data'] !== undefined) {
                        echarts.registerMap(map['name'], map['data'])
                    }
                }
            }
        },
        mounted () {
            /**
             * 获取 ehcarts 挂载元素
             */
            let $echartsDOM = document.getElementById(this.randomId)

            if (!$echartsDOM) return

            let myEcharts = echarts.init($echartsDOM)

            myEcharts.on('click', params => {
                this.echartsClicked(params)
            })
            this.myEcharts = myEcharts

            /**
             * 初始化时进行一次检测
             */
            this.checkPosition()

            /**
             * 对窗口滚动事件进行监控
             */
            window.addEventListener('scroll', this.scrollEvent)

            /**
             * 对浏览器窗口大小改变进行监控
             */
            window.addEventListener('resize', _.throttle(() => {
                this.myEcharts.resize()
            }, this.windowResizeThrottle))
        },
        watch: {

            /**
             * 对 option 的变化进行监控
             */
            option (option) {
                this.checkAndSetOption()
            },

            /**
             * 主动调用 resize 的标识
             */
            resizeSignature () {
                this.myEcharts.resize()
            }
        },
        methods: {
            /**
             * 检测窗口滚动位置
             * 用以进行延迟加载
             */
            checkPosition () {
                let windowHeight = document.documentElement.clientHeight || window.innerHeight
                let scrollTop = document.documentElement.scrollTop || document.body.scrollTop
                let windowBottom = +scrollTop + +windowHeight
                let selfTop = _.get(this.$refs, 'selfEcharts.offsetTop', 0)
                if (windowBottom >= selfTop) {
                    this.isPositionReady = true
                    this.checkAndSetOption()
                    window.removeEventListener('scroll', this.scrollEvent)
                }
            },

            /**
             * 对 option 进行检测
             * 并进行 setOption
             */
            checkAndSetOption () {
                let option = this.option
                if (this.isPositionReady !== true) return
                if (isValidOption(option)) {
                    this.myEcharts.setOption(option, true)
                    this.isOptionAbnormal = false
                } else {
                    this.isOptionAbnormal = true
                }
            },

            /**
             * echarts 点击事件
             * @param params
             */
            echartsClicked (params) {
                this.$emit('echarts-clicked', params)
            }
        }
    }

    /**
     * option 是否合法
     * @param option
     * @returns {boolean}
     */
    function isValidOption (option) {
        return isObject(option) && !isEmptyObject(option) &&
            hasSeriesKey(option) &&
            isSeriesArray(option) && !isSeriesEmpty(option)
    }

    /**
     * option 是否是对象
     * @param option
     * @returns {boolean}
     */
    function isObject (option) {
        return Object.prototype.isPrototypeOf(option)
    }

    /**
     * option 是否是空对象
     * @param option
     * @returns {boolean}
     */
    function isEmptyObject (option) {
        return Object.keys(option).length === 0
    }

    /**
     * option 是否含有 series 键
     * @param option
     * @returns {boolean}
     */
    function hasSeriesKey (option) {
        return !!option['series']
    }

    /**
     * option 中 series 是否是数组
     * @param option
     * @returns {boolean}
     */
    function isSeriesArray (option) {
        return Array.isArray(option['series'])
    }

    /**
     * option 中 series 数组是否是空数组
     * @param option
     * @returns {boolean}
     */
    function isSeriesEmpty (option) {
        return option['series'].length === 0
    }
</script>
