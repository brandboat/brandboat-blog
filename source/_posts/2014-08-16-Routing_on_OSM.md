---
layout: post
title: "Routing On OpenStreetMap (pgrouting)"
description: "postgresql+postGIS+pgrouting+nodejs+OSM"
date: 2014-08-16
categories: [OSM, nodejs, postgresql, postGIS, pgrouting]
---

Demo: http://routingonosm-brandboat.rhcloud.com/
github project: https://github.com/brandboat/Routing-On-OSM

這個 project 主要是利用 pgrouting 讓 openstreetmap 來做兩點最佳路徑規劃，像這類 routing project 其實已經是老生常談，大概 google 一下都能找到很類似的 project ([OSM Routing](http://wiki.openstreetmap.org/wiki/Routing))，但是利用 pgrouting 的似乎不多，google找到相關技術整合的文章也是寥寥可數，我在這邊分享我如何利用 openstreetmap + pgrouting + nodejs 寫出一個簡單的 routing project，以及遇到的問題。

## 介紹
pgrouting 是一套基於 postgresql/postGIS 上的套件，他賦予 `geospatial routing database` `routing` 功能，postgresql 本身是與 MySQL 打對台的關聯式資料庫，但是 postGIS 這個套件讓 postgresql 特別去處理軌跡路線。所以要使用 pgrouting 裏頭的功能你必須先安裝 postgresql 以及 postGIS。

## 安裝
- **postgresql** : `sudo atp-get install postgresql-9.1` (Ubuntu)
當然要安裝別的版本(Ex. 9.3)都沒關係，重點是接下來要安裝的postGIS 一定要是 2.0 以上版本，否則你無法在 `postgresql` 使用 `pgrouting` 以及 `osm2pgrouting`，然而在 Ubuntu 底下當你使用 apt-get 安裝postgresql9.1 他預設使用 postGIS 1.5。所以你必須先移除postGIS(`sudo apt-get remove --purge postgis`)再自己在抓 postGIS 2.0 source code 下來自己安裝。

- **postGIS** : 這邊我建議直接參考 [本篇教學](http://www.jaloonz.com/2012/08/installing-postgis-201-on-ubuntu-1204.html) ，因為 postGIS2.0 使用到很多套件都是 Ubuntu 套件管理中心沒有的，你得自己抓原始碼 compile。 (你還在用 sudo make install 嗎? 試試 sudo checkinstall，至於好處在哪? 直接使用你就知道了)

- **pgrouting** : `sudo apt-get install postgresql-9.1-pgrouting` 
- **osm2pgrouting** : `sudo apt-get install osm2pgrouting` openstreetmap 直接下載的圖資沒辦法直接在pgrouting底下使用，你得先利用此套件轉換。

>為了測試你到底有沒有安裝正確，先進入 database
>`psql -U postgres` or `psql -U postgres -d postgres` (-U: user -d:dbname)
>postgresql 預設建立一位使用者 postgres 以及一個同名資料庫 postgres
>進入之後打: 
>`CREATE EXTENSION postgis;`
>`CREATE EXTENSION pgrouting;`
>如果沒有出現 error 那就沒問題了。如果出現 ` could not open extension control file "/usr/share/postgresql/9.1/extension/postgis.control": No such file or directory`，這就是因為 postGIS 還是1.5版本。
>接下來的使用 pgrouting 方式這邊就不再詳細說明，[pgrouting 官方說明文件](http://workshop.pgrouting.org/index.html)寫得很清楚

-----------

## 應用 (Routing On OpenStreetMap)

## 開發環境
- OS : Ubuntu 12.04 
- nodejs : 0.11.0
- postgresql : 9.1
- postGIS : 2.0
- pgrouting : 9.1(base on postgresql 9.1)

**nodejs 套件**

- [**nodejs**](http://nodejs.org/) + [**pgrouting**](http://pgrouting.org/) + [**openstreetmap**](http://www.openstreetmap.org/) : 
nodejs 使用 MVC 框架的方式有很多，我使用的是 express ，然後參考以下 project 的寫法 [express4-bootstrap-starter](https://github.com/aredo/express4-bootstrap-starter/blob/master/app%2Fconfig%2Fexpress.js)，詳細 nodejs 過程我就不說明了，google 一下其實滿多基礎教學。

- [**node-postgres**](https://github.com/brianc/node-postgres) : nodejs底下與 postgresql 整合的非常好的套件。
- [**leaflet**](http://leafletjs.com/) : An Open-Source JavaScript Library for Mobile-Friendly Interactive Maps
- [**Semantic-UI**](http://semantic-ui.com/) : UI

## 如何使用本 project 
- 安裝好上述套件
- 取得圖資
- 將圖資 load 進 postgresql 
```
osm2pgrouting -file "sampledata.osm" \
                          -conf "/usr/share/osm2pgrouting/mapconfig.xml" \
                          -dbname DBNAME \
                          -user USERNAMWE \
                          -clean
```
- node server.js
- 點擊 Begin 之後任意點擊地圖任何一點，再點 End 重複相同動作之後點選 Route.

## 重點 SQL 語法說明 (app/controllers/route.js)
進入database後，打`\d`就可以看到該database下的所有欄位。
以下是裝好 `postgis` `pgrouting` 並使用 `osm2pgrouting` 就會 自動幫你建立的欄位。

 List of relations 

 Schema |           Name           |   Type   |   Owner   
--------|--------------------------|----------|----------
 public | classes                  | table    | brandboat 
 public | geography_columns        | view     | brandboat 
 public | geometry_columns         | view     | brandboat 
 public | nodes                    | table    | brandboat 
 public | raster_columns           | view     | brandboat 
 public | raster_overviews         | view     | brandboat 
 public | relation_ways            | table    | brandboat 
 public | relations                | table    | brandboat 
 public | spatial_ref_sys          | table    | brandboat 
 public | types                    | table    | brandboat 
 public | way_tag                  | table    | brandboat 
 public | ways                     | table    | brandboat 
 public | ways_vertices_pgr        | table    | brandboat 
 public | ways_vertices_pgr_id_seq | sequence | brandboat 


1. `SELECT id FROM ways_vertices_pgr ORDER BY st_distance(the_geom, st_setsrid(st_makepoint(reqPoint[i].lng + "," + reqPoint[i].lat + "), 4326)) LIMIT 1;`
取得地圖上離reqPoint[i]最近的路的其中一點

2. `WITH result AS (SELECT * FROM ways JOIN (SELECT seq, id1 AS node, id2 AS edge_id, cost, ROW_NUMBER() OVER (PARTITION BY 1) AS rank FROM pgr_dijkstra('SELECT gid AS id, source::integer, target::integer, length::double precision AS cost FROM ways'," + begin + ", " + end + ", false, false)) AS route ON ways.gid = route.edge_id ORDER BY rank) SELECT ST_AsEWKT(result.the_geom), name from result;` 
這段很長，我將他拆開來討論
- `pgr_dijkstra('SELECT gid AS id, source::integer, target::integer, length::double precision AS cost FROM ways'," + begin + ", " + end + ", false, false)`
這段是利用pgr_dijkstra來取得begin和end中的最短路徑。
- `SELECT seq, id1 AS node, id2 AS edge_id, cost, ROW_NUMBER() OVER (PARTITION BY 1) AS rank FROM pgr_dijkstra......ORDER BY rank`
本來我沒有加ROW_NUMBER() OVER (PARTITION BY 1) AS rank，但後來發現，pgrouting他所回傳的路徑並沒有按照順序，這是他的一大缺點...所以我必須額外幫他增加一個欄位來紀錄路徑順序，並按照順序排列。
- `SELECT * FROM ways JOIN...ON ways.gid = route.edge_id`
這段是因為，pgr_dijkstra，回傳的東西裡頭並沒有路徑軌跡點，只有他的id，所以必須再跟ways做join來取得路徑軌跡點
- `WITH result AS...SELECT ST_AsEWKT(result.the_geom), name from result;` 最後將所有得到的結果命為 `result` table，且因為其軌跡直為 linestring，為了再網頁中顯示，必須再將他轉為一般座標點，所以用 `ST_AsEWKT` 將 result 底下存`linestring`的欄位`the_geom`將其轉為一般座標點。

## 經驗談
我很喜歡 `nodejs` ，剛好專題可能需要用到 pgrouting ，想說當作練功，就試著整合看看，不過自己剛踏入 nodejs 領域，對於很多概念還不夠清晰，很多文件看一看都是一知半解，只能透過實作來了解其中原理，特別台灣在nodejs方面文件都只有新手教學，安裝，get post...等，很多進階實現方法還有概念，文件都不多，可能這個領域的人比較喜歡自幹吧!XDDD，相較於RoR，喜歡出來分享相關經驗的有些少。

pgrouting 特別是在他基於 database 之上，你可以透過 query 就直接使用 pgrouting 裏頭提供的功能，相當方便，雖然我知道 routing 很多人都依賴用 google api ，但 google 畢竟不是做慈善，很多東西到一定量之後就要收費，openstreetmap 加上 pgrouting 完全無限制，但是國內卻沒有任何文章分享，實在很可惜...在這邊寫一篇簡單的開發經歷，一方面希望能幫到往後會利用到相關整合的人，如有任何疏漏或者指教之處，麻煩您 email 至 `brandboat@gmail.com`

## 架設平台 
- [openshift](https://www.openshift.com/) **推薦!!!** (但是文件檔不像`heroku`那樣簡潔就是了)
- [masperverpo](http://www.mapserverpro.com/)
- [PostGIS hosters](http://trac.osgeo.org/postgis/wiki/UsersWikiPostGISHosters)
- http://gis.stackexchange.com/questions/25431/any-pgrouting-enabled-hosting-providers

## 疑難雜症
1. `psql -U postgres`  無法使用
Solution : 
- `sudo -u postgres psql` 或
- `sudo vim  /etc/postgresql/9.1/main/pg_hba.conf` 更改
local   all             postgres                            trust
local   all             all                                 trust
host    all             all             127.0.0.1/32        trust 
host    all             all             ::1/128             trust

2. 在使用 `overpass api` 時，我使用以下code取得資料後，使用 osm2pgrouting 將資料 load 進 postgresql，卻發現無法使用...，可能是少了某些重要資料吧，於是我就乖乖用原本 osm api 去拿圖資。

``` 
<!--視覺所及的路段-->
<osm-script output="osm" timeout="25">
  <query type="way">
    <has-kv k="highway"/>
    <bbox-query {{bbox}}/>
  </query>
  <print mode="body"/>
  <recurse type="down"/>
  <print mode="skeleton" order="quadtile" />
</osm-script>
```

這使得我把其他不相關路的資訊也被迫load進圖資裡頭，使得最短路徑受到干擾，
ex.橫跨路中央的橋就會讓結果受到干擾。(待解決)
<a href="http://imgur.com/l8HsoJw"><img src="http://i.imgur.com/l8HsoJw.png?1" title="Hosted by imgur.com" /></a>


參考 : 
http://workshop.pgrouting.org/
https://github.com/aredo/express4-bootstrap-starter/blob/master/app%2Fconfig%2Fexpress.js
https://www.openshift.com/blogs/add-map-navigation-to-your-app-with-pgrouting-on-openshift
https://www.openshift.com/blogs/instant-mapping-applications-with-postgis-and-nodejs

