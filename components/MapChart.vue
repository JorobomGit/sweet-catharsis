<template>
  <div class="map-chart" id="chartdiv">
  </div>
</template>

<script>
import * as am4core from '@amcharts/amcharts4/core';
import * as am4maps from '@amcharts/amcharts4/maps';

import am4themesAnimated from '@amcharts/amcharts4/themes/animated';
import am4themesSpiritedaway from '@amcharts/amcharts4/themes/spiritedaway';
import am4geodataWorldLow from '@amcharts/amcharts4-geodata/worldLow';
import am4geodataDataCountries2 from '@amcharts/amcharts4-geodata/data/countries2';

export default {
  mounted() {
    am4core.ready(() => {
      // Themes begin
      am4core.useTheme(am4themesSpiritedaway);
      am4core.useTheme(am4themesAnimated);
      // Themes end

      const continents = {
        AF: 0,
        AN: 1,
        AS: 2,
        EU: 3,
        NA: 4,
        OC: 5,
        SA: 6,
      };

      // Create map instance
      const chart = am4core.create('chartdiv', am4maps.MapChart);
      chart.projection = new am4maps.projections.Miller();

      // Create map polygon series for world map
      const worldSeries = chart.series.push(new am4maps.MapPolygonSeries());
      worldSeries.useGeodata = true;
      worldSeries.geodata = am4geodataWorldLow;
      worldSeries.exclude = ['AQ'];

      const worldPolygon = worldSeries.mapPolygons.template;
      worldPolygon.tooltipText = '{name}';
      worldPolygon.nonScalingStroke = true;
      worldPolygon.strokeOpacity = 0.5;
      worldPolygon.fill = am4core.color('#eee');
      worldPolygon.propertyFields.fill = 'color';

      let hs = worldPolygon.states.create('hover');
      hs.properties.fill = chart.colors.getIndex(9);

      // Create country specific series (but hide it for now)
      const countrySeries = chart.series.push(new am4maps.MapPolygonSeries());
      countrySeries.useGeodata = true;
      countrySeries.hide();
      countrySeries.geodataSource.events.on('done', () => {
        worldSeries.hide();
        countrySeries.show();
      });

      const countryPolygon = countrySeries.mapPolygons.template;
      countryPolygon.tooltipText = '{name}';
      countryPolygon.nonScalingStroke = true;
      countryPolygon.strokeOpacity = 0.5;
      countryPolygon.fill = am4core.color('#eee');

      hs = countryPolygon.states.create('hover');
      hs.properties.fill = chart.colors.getIndex(9);

      // Set up click events
      worldPolygon.events.on('hit', (ev) => {
        ev.target.series.chart.zoomToMapObject(ev.target);
        const { map } = ev.target.dataItem.dataContext;
        if (map) {
          // eslint-disable-next-line no-param-reassign
          ev.target.isHover = false;
          countrySeries.geodataSource.url = `https://www.amcharts.com/lib/4/geodata/json/${map}.json`;
          countrySeries.geodataSource.load();
        }
      });

      // Set up data for countries
      const data = [];
      // eslint-disable-next-line no-restricted-syntax
      for (const id in am4geodataDataCountries2) {
        // eslint-disable-next-line no-prototype-builtins
        if (am4geodataDataCountries2.hasOwnProperty(id)) {
          const country = am4geodataDataCountries2[id];
          if (country.maps.length) {
            data.push({
              id,
              color: chart.colors.getIndex(continents[country.continent_code]),
              map: country.maps[0],
            });
          }
        }
      }
      worldSeries.data = data;

      // Zoom control
      chart.zoomControl = new am4maps.ZoomControl();

      const homeButton = new am4core.Button();
      homeButton.events.on('hit', () => {
        worldSeries.show();
        countrySeries.hide();
        chart.goHome();
      });

      homeButton.icon = new am4core.Sprite();
      homeButton.padding(7, 5, 7, 5);
      homeButton.width = 30;
      homeButton.icon.path = 'M16,8 L14,8 L14,16 L10,16 L10,10 L6,10 L6,16 L2,16 L2,8 L0,8 L8,0 L16,8 Z M16,8';
      homeButton.marginBottom = 10;
      homeButton.parent = chart.zoomControl;
      homeButton.insertBefore(chart.zoomControl.plusButton);
    }); // end am4core.ready()
  },
};

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  width: 100%;
  height: 500px;
}
</style>
