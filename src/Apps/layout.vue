<template>
    <div>
        <earthViewer></earthViewer>
        <cesiumDrawViewer :extendMarkerImage="imags" ref='marker' :extendMarkerModel="model"></cesiumDrawViewer>
    </div>
</template>

<script>
    import earthViewer from "./cesiumViewer";
    import cesiumDrawViewer from '@/components/cesiumDrawViewer'

    export default {
        data() {
            return {
                drawHelper: "",
                viewerMounted: false,
                drawviewerShow: true,
                imags: [
                    "./static/images/markers/1.png",
                    "./static/images/markers/2.png",
                    "./static/images/markers/3.png",
                    "./static/images/markers/4.png",
                    "./static/images/markers/5.png",
                    "./static/images/markers/6.png",
                    "./static/images/markers/7.png",
                    "./static/images/markers/8.png",
                    "./static/images/markers/5.png",
                    "./static/images/markers/6.png"
                ],
                model: [{id: "model0", name: "木塔", url: "static/model/Wood_Tower.gltf"},
                    {id: "model1", name: "人", url: "static/model/Cesium_Man.gltf"}]

            };
        },
        components: {
            earthViewer,
            cesiumDrawViewer,
        },
        props: {},
        computed: {},
        beforeMount() {
        },
        mounted() {
            // self.viewer = window.cesiumViewer;
            const viewer = window.cesiumViewer;
            const tileset = new Cesium.Cesium3DTileset({
                url: 'static/temp/tileset.json',
                maximumMemoryUsage: 100,//不可设置太高，目标机子空闲内存值以内，防止浏览器过于卡
                maximumScreenSpaceError: 20,//用于驱动细节细化级别的最大屏幕空间错误;较高的值可提供更好的性能，但视觉质量较低。
                maximumNumberOfLoadedTiles: 1000,  //最大加载瓦片个数
                shadows: false,//是否显示阴影
                skipLevelOfDetail: true,
                baseScreenSpaceError: 1024,
                skipScreenSpaceErrorFactor: 16,
                skipLevels: 1,
                immediatelyLoadDesiredLevelOfDetail: false,
                loadSiblings: false,
                cullWithChildrenBounds: true,
                dynamicScreenSpaceError: true,
                dynamicScreenSpaceErrorDensity: 0.00278,
                dynamicScreenSpaceErrorFactor: 4.0,
                dynamicScreenSpaceErrorHeightFalloff: 0.25
            })
            this.$refs.marker.init(viewer);
            tileset.readyPromise.then(t => {
                var height = 15;
                height = Number(height);
                if (isNaN(height)) {
                    return;
                }
                var cartographic = Cesium.Cartographic.fromCartesian(t.boundingSphere.center);
                var surface = Cesium.Cartesian3.fromRadians(cartographic.longitude, cartographic.latitude, cartographic.height);
                var offset = Cesium.Cartesian3.fromRadians(cartographic.longitude, cartographic.latitude, height);
                var translation = Cesium.Cartesian3.subtract(offset, surface, new Cesium.Cartesian3());
                t.modelMatrix = Cesium.Matrix4.fromTranslation(translation);

                viewer.scene.primitives.add(t)
                viewer.camera.viewBoundingSphere(t.boundingSphere)
            })
        },
        methods: {},
        watch: {}
    };
</script>

<style scoped>
</style>
<style>
    /*定义滚动条高宽及背景 高宽分别对应横竖滚动条的尺寸*/
    ::-webkit-scrollbar {
        width: 5px;
        height: 5px;
        background-color: #e0e0e0;
    }

    /*定义滚动条轨道 内阴影+圆角*/
    ::-webkit-scrollbar-track {
        -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
        border-radius: 5px; /*滚动条的背景区域的圆角*/
        background-color: #a1a1a1; /*滚动条的背景颜色*/
    }

    /*定义滑块 内阴影+圆角*/
    ::-webkit-scrollbar-thumb {
        border-radius: 5px; /*滚动条的圆角*/
        -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
        background-color: #e0e0e0; /*滚动条的背景颜色*/
    }

    #toolbar {
        width: 100px;
        height: 200px;
        position: fixed;
        top: 10px;
        left: 10px;
        background-color: whitesmoke;
    }

    .toolbar {
        position: absolute;
        top: 50px;
        right: 50px;
        z-index: 100;
    }

    .toolbar .el-button {
        background-color: #ffffff00;
        border: 0px;
        margin-left: 0px;
    }
</style>
