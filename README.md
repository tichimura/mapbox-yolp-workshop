# YOLPからMapboxへのマイグレーション・ワークショップ

## 事前準備

### アカウントの作成
https://account.mapbox.com/auth/signup

### TOKENの取得

#### Mapbox(TOKEN)

https://account.mapbox.com/access-tokens/create

#### YOLP(アプリケーションID)

https://e.developer.yahoo.co.jp/register

### Map Style

こちらのサンプル
- mapbox://styles/tichimura/ckf4bl24g1dfi19mv7ksq42ac

あるいは、以下のPublic テンプレートから利用
- mapbox://styles/mapbox/streets-v11
- mapbox://styles/mapbox/outdoors-v11
- mapbox://styles/mapbox/light-v10
- mapbox://styles/mapbox/dark-v10
- mapbox://styles/mapbox/satellite-v9
- mapbox://styles/mapbox/satellite-streets-v11

## 参考情報

- [YOLPでのサンプル](https://gist.githubusercontent.com/tichimura/535313e24b7d32c8a303acace27532f9/raw/ef332216ef00831ca251dda7fbbb3ccc4d6664a6/yolpsample.html)
- [Mapboxでのサンプル](https://gist.githubusercontent.com/tichimura/535313e24b7d32c8a303acace27532f9/raw/ef332216ef00831ca251dda7fbbb3ccc4d6664a6/yolpwebinar.html)
- [APIリファレンス(English)](https://docs.mapbox.com/mapbox-gl-js/api/map)
- [サンプルコード](https://docs.mapbox.com/jp/mapbox-gl-js/example/)

## コード実装

### CDNの設定

```html
<script src='https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.js'></script>
<link href='https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css' rel='stylesheet' />
```

### HTML Style

```html
    <style>
        body {margin: 0; padding: 0;}
        #map { position: absolute; width: 100%; height: 100vh;}
    </style>
<body>
       <div id="map"></div>

       ...(snip)...
```

### 地図を表示する

- YOLPでは自動的にposition:relative、overflow:hiddenといったスタイルが適用される
- 標準地図では1～20まで指定でき、数値が大きくなるほど詳細な縮尺になります。
  - Mapboxでは1-22
  - yolpのズームレベル14 -> Mapboxのズームレベル12あたり
- heightの指定が必要　`#map { width: 100%; height: 100vh;}`

YOLP
```javascript

var ymap = new Y.Map("map");
ymap.drawMap(new Y.LatLng(35.66572, 139.73100), 14, Y.LayerSetId.NORMAL);

```

Mapbox
```javascript
/* 地図を表示 */
var map = new mapboxgl.Map({
  /* 地図を表示させる要素のid */
  container: 'map',
  /* 地図styleID。YOLPではLayerSetIdに相当する。*/
  style: 'mapbox://styles/mapbox/streets-v11',
  /* 地図の初期緯度経度[lng,lat] */
  center: [139.72116702175174, 35.64997652994234],
  /* 地図の初期ズームレベル */
  zoom: 12
});

```

### コントロールの追加

YOLP
```javascript
        var control = new Y.ZoomControl();
        ymap.addControl(control);

```

Mapbox
```javascript
        map.addControl(new mapboxgl.NavigationControl());
```

-> コントロールの種類

### マーカーの追加

YOLP
```javascript
       var marker = new Y.Marker(new Y.LatLng(35.64997652994234,139.72116702175174));
        ymap.addFeature(marker);
```

Mapbox
```javascript
        var marker = new mapboxgl.Marker()
            .setLngLat([139.72116702175174,35.64997652994234])
            .addTo(map);    // 忘れずに
```

https://docs.mapbox.com/mapbox-gl-js/api/markers/

### アイコンの追加

YOLP

```javascript
var icon = new Y.Icon('https://s.yimg.jp/images/map/yy/images/icon/yy_aicon_06_32pix.gif');
var marker = new Y.Marker(new Y.LatLng(35.64997652994234,139.72116702175174), {icon: icon});
ymap.addFeature(marker);
```

Mapbox

1. DOMで追加

```javascript
        var el = document.createElement('div'); // new しない
        el.id = 'marker';
        el.style.width = '32px';
        el.style.height = '40px';
        el.style.backgroundImage = 'url(https://i.imgur.com/MK4NUzI.png)';
        var marker = new mapboxgl.Marker(el)
            .setLngLat([139.72116702175174,35.64997652994234])            
            .addTo(map);
```

2. addImage


```javascript
            map.loadImage('https://i.imgur.com/MK4NUzI.png', function(error, image) {
                // if (error) throw error;
                if (!map.hasImage('mapbox_icon')) map.addImage('mapbox_icon', image);
                map.addLayer({
                    'id': 'mapboxicon',
                    'type': 'symbol',
                    'source': {
                        type: 'geojson',
                        data: {
                            type: 'Feature',
                            geometry:{
                                type: 'Point',
                                coordinates:[
                                    139.72116702175174, 35.64997652994234
                                ]
                            },
                            properties:{
                                "title": "YOLP DEMO ICON"
                            }
                        }
                    },
                    layout:{
                        'icon-image': 'mapbox_icon',
                        'icon-size': 0.5
                    }
                })
            });

```

https://docs.mapbox.com/mapbox-gl-js/style-spec/layers/#layout-symbol-icon-size


#### (参考) loadImage(url, callback)では、URLで指定した外部ドメインはCORSをサポートしている必要あり
` map.loadImage('https://s.yimg.jp/images/map/yy/images/icon/yy_aicon_06_32pix.gif', function(error, image) {
`

```error
Access to fetch at 'https://s.yimg.jp/images/map/yy/images/icon/yy_aicon_06_32pix.gif' from origin 'http://localhost:8000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.
ajax.js:148 GET https://s.yimg.jp/images/map/yy/images/icon/yy_aicon_06_32pix.gif net::ERR_FAILED
```

### ラベルの追加

YOLP

```javascript
var label = new Y.Label(new Y.LatLng(35.64997652994234,139.72116702175174), "打ち上げ会場");
ymap.addFeature(label);
```

Mapbox

```javascript
var label = new mapboxgl.Popup().setLngLat([139.72116702175174,35.64997652994234]).setText("打ち上げ会場").addTo(map);
```

**`addTo(map)` を忘れずに!**

### ポップアップ(吹き出し)

YOLP

```javascript
map.drawMap(new Y.LatLng(35.680840,139.767009), 10, Y.LayerSetId.NORMAL);
map.openInfoWindow(map.getCenter(), "ここは東京駅です。");
```

クリックしてポップアップを呼ぶ

```javascript
marker.bind('click', function(){
        ymap.openInfoWindow(marker.getLatLng(), "ここは打ち上げ会場です。");
});
```

Mapbox

Markerインスタンスに付与する（Markerインスタンスがある、clickイベントで呼ぶ場合）

```javascript
var marker = new mapboxgl.Marker(el).setLngLat([139.72116702175174, 35.64997652994234])
    .setPopup(new mapboxgl.Popup().setText("打ち上げ会場"))
    .addTo(map);
```


(ラベルと同じ) Popupインスタンスを立ち上げる（Markerインスタンスがない、Symbol Layerがあるときなど、あるいはイベントなしで表示したい場合）
```javascript
new mapboxgl.Popup().setLngLat([139.72116702175174,35.64997652994234])setHTML("<h1>Hello World!</h1>").addTo(map)
```


### イベント処理

YOLP
```javascript
        ymap.bind('click', function(latlng){
           alert('クリックされた場所は '+latlng.toString());
        });
```

Mapbox
```javascript
        map.on('click', function(e) {
            alert('クリックされた場所は ' + e.lngLat.lat + ',' + e.lngLat.lng);
        });
```

-> イベントの種類

### 中心位置の移動(PanToの追加)

(イベント処理との組み合わせ)

YOLP
```javascript
        window.onload = function() {
            ymap.panTo(new Y.LatLng(35.67961620200805,139.7369242375451), true);
        }
```

Mapbox
```javascript
        map.on('click', function(){
            alert('クリックされた場所へ移動します ' + e.lngLat.lat + ',' + e.lngLat.lng);
            var ll = e.lngLat;
            // var ll = new mapboxgl.LngLat(e.lngLat.lng, e.lngLat.lat);
            map.panTo(ll)
            // marker.setLngLat(ll);            
        });
```


### ポリゴンの追加(線、円)

#### 線

YOLP
```javascript
var style = new Y.Style("00ff00", 8, 0.5);
var latlngs = [
   new Y.LatLng(35.650876676313125,139.72215675687852),
   new Y.LatLng(35.65078295694602,139.72216212129663),
   new Y.LatLng(35.650713212229476,139.72212188816138),
   new Y.LatLng(35.65043205322328,139.72116165733408),
   new Y.LatLng(35.64999832537358,139.72135209417374)
];
var polyline = new Y.Polyline(latlngs, {strokeStyle: style});
ymap.addFeature(polyline);

```

Mapbox

**map.on('load', function(){})内で定義する**

```javascript

map.addLayer({
id: "demo-line",
source:{
        type: "geojson",
        data: {
        type: "Feature",
        geometry: {
                "type": "LineString",
                "coordinates":[
                [139.72222056707562, 35.65229501202714],
                [139.72215675687852, 35.650876676313125],
                [139.72216212129663, 35.65078295694602],
                [139.72212188816138, 35.650713212229476],
                [139.72116165733408, 35.65043205322328],
                [139.72135209417374, 35.64999832537358]
                ]
        },
        properties:{
                "title": "YOLP DEMO LINE"
        }
        }

},
type: 'line',
paint:{
        'line-color': "#00ff00",
        'line-width': 8,
        'line-opacity':0.5
}
})

```

#### 円

YOLP

```javascript
var strokeStyle = new Y.Style("00ff00", 4, 0.7);
var fillStyle   = new Y.Style("00ff00", null, 0.2);
var circle = new Y.Circle(new Y.LatLng(35.64997652994234,139.72116702175174), new Y.Size(100, 100), {
   strokeStyle: strokeStyle,
   fillStyle:fillStyle
});
ymap.addFeature(circle);
```

Mapbox

```javascript
            map.addLayer({
                id: "demo-circle",
                source:{
                    type: "geojson",
                    data: {
                        type: "Feature",
                        geometry: {
                            "type": "Point",
                            "coordinates":[
                                139.72116702175174, 35.64997652994234
                            ]
                        },
                        properties:{
                            "title": "YOLP DEMO CIRCLE"
                        }
                        
                    }

                },
                type: 'circle',
                paint:{
                    'circle-radius': 100,
                    'circle-color': "#00ff00",
                    'circle-stroke-width': 4,
                    'circle-stroke-color': "#00ff00",
                    'circle-opacity':0.2
                }
            })
```


### ポリゴンの削除(線、円)

YOLP
```javascript
ymap.removeFeature(circle);
```

Mapbox

```javascript
map.removeLayer('demo-circle');
```

### 外部データの取り込み（YDF/GeoJSON)

YOLP

```javascript
 var xmlLayer = new Y.GeoXmlLayer("https://s.yimg.jp/images/map/yolp/developer_network/ydf/geoSearch.xml");
 map.addLayer( xmlLayer );
 xmlLayer.execute();
```

Mapbox

```javascript
map.addSource('poi-source', {
        'type': 'geojson',
        'data': 'https://gist.githubusercontent.com/tichimura/535313e24b7d32c8a303acace27532f9/raw/d1e842a712381ca74d15c71816b1824168d6bfba/yolp.geojson'
});

map.addLayer({
        id: 'poi-layer',
        'type': 'symbol',
        'source':'poi-source',
        'layout': {
                'icon-image': 'restaurant-noodle-15'
        }
});

maki icon: https://labs.mapbox.com/maki-icons/editor/

```

### レイヤーの追加・削除

YOLP
```javascript
map.removeLayer(xmlLayer);
```

Mapbox
```javascript

map.on('click', 'poi-layer', function(){
    map.removeLayer('poi-layer');
})
```

#### おまけ

Mapbox
```javascript
        map.on('click', 'poi-layer', function(e){
            new mapboxgl.Popup()
                .setLngLat(e.lngLat)    // lngLatに注意！
                .setHTML(e.features[0].properties.name)
                .addTo(map) // 忘れずに
        })
```

### スタイルの変更（衛星写真にする）

YOLP
```javascript
ymap.setLayerSet(Y.LayerSetId.PHOTO);
```

Mapbox


```javascript
map.setStyle("mapbox://styles/mapbox/satellite-v9");
```

```html

    <style>
    // 追加
        #menu{
            position: absolute;
            background-color: white;
        }
    </style>

    <div id="menu">
        <input id="streets" type="radio" name="stylemenu" checked="checked"/>
        <label for="streets">地図</label>
        <input id="satellite" type="radio" name="stylemenu"/>
        <label for="satellite">衛星画像</label>
    </div>

```

```javascript
    var layerList = document.getElementById('menu');
    var inputs = layerList.getElementsByTagName('input');

    // スタイルが増えたらここを編集する
    function swithLayer(layer){
        var layerId = layer.target.id;
        if(layerId === "streets"){
            map.setLayoutProperty('mapbox-satellite', 'visibility', 'none');
            // map.setStyle("mapbox://styles/tichimura/ckf36b4wa0dnj19nyx7gcfpm2/draft");
        }else if (layerId === "satellite"){
            map.setLayoutProperty('mapbox-satellite', 'visibility', 'visible');
            // map.setStyle("mapbox://styles/mapbox/satellite-v9");
        }
    }

    for (var i = 0; i < inputs.length; i++) {
        inputs[i].onclick = switchLayer;
    }
```


