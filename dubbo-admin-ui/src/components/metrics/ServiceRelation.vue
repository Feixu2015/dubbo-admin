<!--
  - Licensed to the Apache Software Foundation (ASF) under one or more
  - contributor license agreements.  See the NOTICE file distributed with
  - this work for additional information regarding copyright ownership.
  - The ASF licenses this file to You under the Apache License, Version 2.0
  - (the "License"); you may not use this file except in compliance with
  - the License.  You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
  <v-container grid-list-xl fluid>
    <v-layout row wrap>
      <v-flex lg12>
        <breadcrumb title="serviceRelation" :items="breads"></breadcrumb>
      </v-flex>
    </v-layout>

    <v-flex lg12>
      <v-card>
        <div id="chartContent" style="width:100%;height:600%;" />
      </v-card>
    </v-flex>
  </v-container>
</template>
<script>
  import Breadcrumb from "@/components/public/Breadcrumb";
  export default {
    components: {
      Breadcrumb
    },
    data: () => ({
      success: null,
      breads: [
        {
          text: "serviceMetrics",
          href: ""
        },
        {
          text: "serviceRelation",
          href: ""
        }
      ],
      responseData: null
    }),
    methods: {
      initData: function() {
        // eslint-disable-next-line no-undef
        this.chartContent = echarts.init(document.getElementById("chartContent"));
        this.chartContent.showLoading();
        this.$axios
          .get("/metrics/relation")
          .then(response => {
            if (response && response.status === 200) {
              this.success = true;
              this.responseData = response.data;
              this.responseData.type = "force";
              this.initChart(this.responseData);
            }
          })
          .catch(error => {
            this.success = false;
            this.responseData = error.response.data;
          });
      },
      initChart: function(data) {
        this.chartContent.hideLoading();
        const links = [...this.responseData.links];
        const colors = [
          'Black',
          'Blue',
          'BlueViolet',
          'Brown',
          'BurlyWood',
          'Chocolate',
          'Coral',
          'Crimson',
          'Cyan',
          'DeepPink',
          'DeepSkyBlue',
          'DimGray',
          'DodgerBlue',
          'Feldspar',
          'FireBrick',
          'FloralWhite',
          'ForestGreen',
          'Fuchsia',
          'Gainsboro',
          'GhostWhite',
          'Gold',
          'GoldenRod',
          'Gray',
          'Green',
          'GreenYellow',
          'HoneyDew',
          'HotPink',
          'IndianRed',
          'Indigo',
          'Ivory',
          'Khaki',
          'Lavender',
          'LavenderBlush',
          'LawnGreen',
          'LemonChiffon',
          'Lime',
          'LimeGreen',
          'Linen',
          'Magenta',
          'Maroon',
          'MidnightBlue',
          'MintCream',
          'MistyRose',
          'Moccasin',
          'Navy',
          'OldLace',
          'Olive',
          'OliveDrab',
          'Orange',
          'OrangeRed',
          'Orchid',
          'PaleGoldenRod',
          'PaleGreen',
          'PaleTurquoise',
          'PaleVioletRed',
          'PapayaWhip',
          'PeachPuff',
          'Peru',
          'Pink',
          'PowderBlue',
          'Purple',
          'Red',
          'RosyBrown',
          'RoyalBlue',
          'SaddleBrown',
          'SandyBrown',
          'SeaGreen',
          'SkyBlue',
          'SlateBlue',
          //'Snow',
          'SpringGreen',
          'SteelBlue',
          'VioletRed',
          'Yellow'
        ]
        const random = (min, max) => {
          const imin = Math.ceil(min);
          const imax = Math.floor(max);
          return Math.floor(Math.random() * (imax - imin)) + imin;
        }
        const doubled = [];
        data.nodes.forEach((node, index) => {
          const old = node.index;
          links.forEach((link, li) => {
            if (!link.sourceChanged && link.source === old) {
              link.source = index
              link.sourceChanged = true
            }
            if (!link.targetChanged && link.target === old) {
              link.target = index
              link.targetChanged = true
            }
            let curveness = 0;
            if (-1 !== doubled.findIndex(it => it === `${link.target}_${link.source}`)) {
              curveness = 0.3;
            }
            link.lineStyle = {
              color: colors[random(0, data.nodes.length)],
              width: 1,
              curveness: curveness
            }
            doubled.push(`${link.source}_${link.target}`);
          })
          node.index = index
        })
        console.log('links:', links);
        const option = {
          legend: {
            data: data.categories.map(i => i.name)
          },
          animationDurationUpdate: 1000,
          animationEasingUpdate: "quinticInOut",
          series: [
            {
              type: "graph",
              layout: "force",
              symbolSize: 20,
              draggable: true,
              label: {
                show: true
              },
              force: {
                repulsion : 100,//节点之间的斥力因子。支持数组表达斥力范围，值越大斥力越大。
                gravity : 0.03,//节点受到的向中心的引力因子。该值越大节点越往中心点靠拢。
                edgeLength :100,//边的两个节点之间的距离，这个距离也会受 repulsion。[10, 50] 。值越大则长度越长
                layoutAnimation : true
              },
              edgeSymbol: ["", "arrow"],
              data: data.nodes.map(function(node, idx) {
                node.id = idx;
                return node;
              }),
              categories: this.responseData.categories,
              links
            }
          ]
        };
        this.chartContent.setOption(option);
      }
    },
    mounted: function() {
      this.initData();
    }
  };
</script>
