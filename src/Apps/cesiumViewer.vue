<template>
  <div style="width:100%;height: 100%" class="fullSize">
    <div class="full-container" :style="viewStyle" id="cesiumContainer"></div>
    <div id="loadingOverlay">
      <h1>Loading...</h1>
    </div>
  </div>
</template>

<script>
export default {
  name: "EarthViewer",
  viewerProperty: {},
  props: {
    viewStyle: {},
    viewerProperty: {},
    dropBackground: {
      default: false
    }
  },
  components: {},
  data() {
    return {
      viewer: null,
        classi: {
            ld: [
                {
                    id: "发电站",
                    coordinates: [
                        -75.59669570330735,40.03767518895486,    // * 关键控制点：顺时针方向 （修改）
                        -75.5973086782783,40.038645198914494,
                        -75.59576098154865,40.03917752557398,
                        -75.59528311684002,40.03861379983191,
                    ],
                    height: 1000,   // * 海拔 （修改）
                }
            ]
        }
    };
  },
  computed: {},
  mounted() {
    const _this = this;
    _this.viewerDefaultProperty = {
      animation: false,
      timeline: false,
      geocoder: false,
      homeButton: false,
      navigationHelpButton: false,
      baseLayerPicker: false,
      fullscreenElement: "cesiumContainer",
      fullscreenButton: false,
      shouldAnimate: true,
      infoBox: false,
      selectionIndicator: false,
      sceneModePicker: false,
      shadows: false,
      skyAtmosphere: false,
      imageryProvider: new Cesium.UrlTemplateImageryProvider({
        url: "http://mt1.google.com/vt/lyrs=s&hl=zh-CN&x={x}&y={y}&z={z}&s=Gali"
      })
    };

    for (let property in _this.viewerProperty) {
      _this.viewerDefaultProperty[property] = _this.viewerProperty[property];
    }

    const viewer = new Cesium.Viewer("cesiumContainer", {
      animation: false,
      timeline: false,
      geocoder: false,
      homeButton: false,
      navigationHelpButton: false,
      baseLayerPicker: false,
      fullscreenElement: "cesiumContainer",
      fullscreenButton: false,
      shouldAnimate: true,
      infoBox: false,
      selectionIndicator: false,
      sceneModePicker: false,
      shadows: false,
      skyAtmosphere: false,
      imageryProvider: new Cesium.UrlTemplateImageryProvider({
        url: "http://mt1.google.com/vt/lyrs=s&hl=zh-CN&x={x}&y={y}&z={z}&s=Gali",
        tilingScheme: new Cesium.WebMercatorTilingScheme(),
        minimumLevel: 1,
        maximumLevel: 20
      })
    });
    window.cesiumViewer = viewer;

    //清除cesium-widget-credits
    const credits = document.getElementsByClassName("cesium-widget-credits");
    credits[0].parentElement.removeChild(credits[0]);

    //禁止双击zoom
    viewer.cesiumWidget.screenSpaceEventHandler.removeInputAction(
      Cesium.ScreenSpaceEventType.LEFT_DOUBLE_CLICK
    );
    viewer.scene.postProcessStages.fxaa.enabled = false;
    viewer.camera.flyTo({
      destination: Cesium.Cartesian3.fromDegrees(116.4, 39.9, 15000000),
      duration: 0
    });
    // viewer.scene.camera.setView({
    //   destination: new Cesium.Cartesian3(
    //     277096.634865404,
    //     5647834.481964232,
    //     2985563.7039122293
    //   ),
    //   orientation: {
    //     heading: 4.731089976107251,
    //     pitch: -0.32003481981370063
    //   }
    // });

      var handler = new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas);
      handler.setInputAction(event => {
          var cartesian = viewer.scene.pickPosition(event.position);
          //输出之后我们发现如前言所说的坐标都是笛卡尔坐标，所以我们需要转换笛卡尔坐标
          console.log("笛卡尔3："+cartesian);
          console.log("高度="+cartesian.height)
          // 屏幕坐标转为空间坐标
          console.log(event.position)
          //let cartesian = viewer.camera.pickEllipsoid(event.position, viewer.scene.globe.ellipsoid);
          if (cartesian == undefined) {
              console.log('没有获取到坐标')
          } else {
              // 空间坐标转世界坐标(弧度)
              let cartographic = Cesium.Cartographic.fromCartesian(cartesian);
              console.log()
              // 弧度转为角度（经纬度）
              console.log(cartographic)
              let lon = Cesium.Math.toDegrees(cartographic.longitude);  // 经度值
              let lat = Cesium.Math.toDegrees(cartographic.latitude);   // 纬度值

              console.log('经纬度是：', { x: lon, y: lat });
          }
      }, Cesium.ScreenSpaceEventType.LEFT_CLICK)
      //this.single()
      this.classification();
      this.handlerMouseMove();
  },
  methods: {
      single(){
          var center = {
              x: 1216368.1741733507,
              y: -4736283.027910914,
              z: 4081277.4451958854
          }
         /* var car33 = Cesium.Cartesian3.fromDegrees(point.lng, point.lat, point.alt);
          var x = car33.x;
          var y = car33.y;
          var z = car33.z;*/
         let modelMatrix = Cesium.Transforms.eastNorthUpToFixedFrame(center);
         let  hprRotation = Cesium.Matrix3.fromHeadingPitchRoll(new Cesium.HeadingPitchRoll(5.785339046755887, 0.0, 0.0));
          let hpr = Cesium.Matrix4.fromRotationTranslation(hprRotation, new Cesium.Cartesian3(-0.25, 0.0, -2.0));
          Cesium.Matrix4.multiply(modelMatrix, hpr, modelMatrix);

          /*var treeHighlight2 = window.cesiumViewer.scene.primitives.add(new Cesium.ClassificationPrimitive({
              geometryInstances : new Cesium.GeometryInstance({
                  geometry : new Cesium.EllipsoidGeometry({
                      radii : new Cesium.Cartesian3(100, 100, 100)
                  }),
                  modelMatrix : modelMatrix,
                  attributes : {
                      color : Cesium.ColorGeometryInstanceAttribute.fromColor(Cesium.Color.fromCssColorString('#F03A47').withAlpha(0.5)),
                      show : new Cesium.ShowGeometryInstanceAttribute(true)
                  },
                  id : 'volume 2'
              }),
              classificationType : Cesium.ClassificationType.CESIUM_3D_TILE
          }));*/
         /* window.cesiumViewer.scene.primitives.add(new Cesium.ClassificationPrimitive({
              geometryInstances: new Cesium.GeometryInstance({
                  geometry: new Cesium.PolygonGeometry({
                      polygonHierarchy: new Cesium.PolygonHierarchy(
                          Cesium.Cartesian3.fromDegreesArray([-75.59656610613676,40.03889085599216])
                      ),
                      extrudedHeight: 2000
                  }),
                  attributes: {
                      color : Cesium.ColorGeometryInstanceAttribute.fromColor(Cesium.Color.fromCssColorString('#F03A47').withAlpha(0.5)),
                      show: new Cesium.ShowGeometryInstanceAttribute(true)
                  },
                  id: 'asdf'
              }),
              classificationType: Cesium.ClassificationType.CESIUM_3D_TILE
          }))*/


          var points = [
              Cesium.Cartesian3.fromDegrees(-75.59704652603575,40.03892589028592),
              Cesium.Cartesian3.fromDegrees(-75.59632568159155,40.039231403008884),
              Cesium.Cartesian3.fromDegrees(-75.59684814944326,40.03864129251899),
              Cesium.Cartesian3.fromDegrees(-75.59612366532117,40.03891619184536)
          ];

          var pointsLength = points.length;
          var clippingPlanes = []; // 存储ClippingPlane集合
          for (var i = 0; i < pointsLength; ++i) {
              var nextIndex = (i + 1) % pointsLength;
              var midpoint = Cesium.Cartesian3.add(points[i], points[nextIndex], new Cesium.Cartesian3());
              midpoint = Cesium.Cartesian3.multiplyByScalar(midpoint, 0.5, midpoint);

              var up = Cesium.Cartesian3.normalize(midpoint, new Cesium.Cartesian3());
              var right = Cesium.Cartesian3.subtract(points[nextIndex], midpoint, new Cesium.Cartesian3());
              right = Cesium.Cartesian3.normalize(right, right);

              var normal = Cesium.Cartesian3.cross(right, up, new Cesium.Cartesian3());
              normal = Cesium.Cartesian3.normalize(normal, normal);

              var originCenteredPlane = new Cesium.Plane(normal, 0.0);
              var distance = Cesium.Plane.getPointDistance(originCenteredPlane, midpoint);

              clippingPlanes.push(new Cesium.ClippingPlane(normal, distance));
          }
          window.cesiumViewer.scene.globe.clippingPlanes = new Cesium.ClippingPlaneCollection({
              planes:clippingPlanes,
              edgeWidth: 4.0,
              edgeColor: Cesium.Color.YELLOW
          });


      },
      classification(){
          let ldCollection = new Cesium.PrimitiveCollection();
          window.ldCollection = ldCollection;
          window.cesiumViewer.scene.primitives.add(ldCollection);
          this.classi.ld.map((value) => {
              let classificationPrimitive = this.addPrimitive(value);
              ldCollection.add(classificationPrimitive);
          });
      },
      addPrimitive(e){
          let classificationPrimitive = new Cesium.ClassificationPrimitive({
              geometryInstances: new Cesium.GeometryInstance({
                  // 多边形对象
                  geometry: new Cesium.PolygonGeometry({
                      polygonHierarchy: new Cesium.PolygonHierarchy(        // hierarchy层次
                          Cesium.Cartesian3.fromDegreesArray(e.coordinates)   // 多边形控制点的经纬度，按顺时针排列
                      ),
                      extrudedHeight: e.height,    // extruded 压制
                      //vertexFormat: Cesium.EllipsoidSurfaceAppearance.VERTEX_FORMAT
                  }),
                  //顶点着色器属性
                  attributes: {
                      color: Cesium.ColorGeometryInstanceAttribute.fromColor(
                          new Cesium.Color(1, 1, 1, 1e-4)
                      ),
                      show: new Cesium.ShowGeometryInstanceAttribute(true), //确定是否显示几何实例
                  },

                  id: e.id,
              }),

              classificationType: Cesium.ClassificationType.BOTH, //是否影响地形
          })
          return classificationPrimitive;
      },
      handlerMouseMove(){
          let viewer = window.cesiumViewer;
          // alert('经过')
          let selected, primitive, color, show, attribute;
          new Cesium.ScreenSpaceEventHandler(viewer.scene.canvas).setInputAction(
              (move) => {
                  let pick = viewer.scene.pick(move.endPosition);

                  if (Cesium.defined(pick) && Cesium.defined(pick.id)) {
                      if (selected === pick.id) return;
                      Cesium.defined(selected) &&
                      (((attribute = primitive.getGeometryInstanceAttributes(
                          selected
                      )).color = color),
                          (attribute.show = show),
                          (selected = void 0),
                          (primitive = void 0),
                          (color = void 0),
                          (show = void 0));
                  }

                  Cesium.defined(pick) &&
                  Cesium.defined(pick.primitive) &&
                  Cesium.defined(pick.id) &&
                  Cesium.defined(pick.primitive.getGeometryInstanceAttributes)
                      ? ((selected = pick.id),
                          (primitive = pick.primitive), // 映射对象
                          (attribute = primitive.getGeometryInstanceAttributes(selected)), // 对象属性
                          (color = attribute.color), // 颜色属性
                          (show = attribute.show), // 显示属性
                      viewer.scene.invertClassification ||
                      (attribute.color = [255, 0, 255, 128]),
                          (attribute.show = [1]))
                      : Cesium.defined(selected) &&
                      (((attribute = primitive.getGeometryInstanceAttributes(
                          selected
                      )).color = color),
                          (attribute.show = show),
                          (selected = void 0),
                          (primitive = void 0),
                          (color = void 0));
              },
              Cesium.ScreenSpaceEventType.MOUSE_MOVE
          );
      }

  }
};
</script>

<style scoped>
.fullSize,
.full-container {
  position: absolute;
  /*top: 0;*/
  /*left: 0;*/
  border: none;
  height: 100%;
  width: 100%;
  margin: 0px;
  display: inherit;
}
.doubleViewer {
  width: 50%;
}
/*#cesiumContainer {*/
/*overflow: hidden;*/
/*position: fixed;*/
/*background: url('../../static/images/sky.jpg') no-repeat;*/
/*margin: 0px;*/
/*background-size: cover;*/
/*}*/

#loadingOverlay {
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0.9;
  width: 100%;
  height: 100%;
  display: none;
}

#loadingOverlay h1 {
  text-align: center;
  position: relative;
  top: 50%;
  margin-top: -0.5em;
}

#mousePositionId {
  position: absolute;
  right: 30px;
  bottom: 50px;
  z-index: 100;
  font-size: 20px;
}
.layer-picker-class {
  float: right;
}
</style>
<style>
html {
  overflow-x: hidden;
  overflow-y: hidden;
}
</style>
