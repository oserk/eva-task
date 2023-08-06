<template>
  <div id="container"></div>
</template>

<script>
var Highcharts = require('highcharts');
require('highcharts/modules/exporting')(Highcharts);

export default {
  props: {
    chartData: {
      type: Object,
    },
  },
  data() {
    return {
      selectedPoints: [],
    };
  },
  mounted() {
    const chartOptions = {
      chart: {
        type: 'column',
      },
      title: {
        text: 'Daily Sales',
        align: 'left',
      },
      xAxis: {
        categories: [],
      },
      yAxis: {
        min: 0,
        title: {
          text: `Amount (${this.chartData.Currency})`,
        },
        stackLabels: {
          enabled: true,
        },
      },
      legend: {
        align: 'left',
        x: 70,
        verticalAlign: 'top',
        y: 70,
        floating: true,
        backgroundColor:
          Highcharts.defaultOptions.legend.backgroundColor || 'white',
        shadow: false,
      },
      tooltip: {
        headerFormat: '<b>{point.x}</b><br/>',
        pointFormat: '{series.name}: {point.y}<br/>Total: {point.stackTotal}',
      },
      plotOptions: {
        series: {
          allowPointSelect: true,
        },
        column: {
          stacking: 'normal',
          dataLabels: {
            enabled: true,
          },
          point: {
            events: {
              click: (event) => {
                if (this.selectedPoints.length > 1) {
                  for (let selectedPoint of this.selectedPoints) {
                    selectedPoint.select(false);
                  }

                  this.selectedPoints = [];
                }

                this.selectedPoints.push(event.point);
                const selectedDates = this.selectedPoints.map(
                  (selectedPoint) => selectedPoint.category
                );
                this.$emit('setSelectedDate', selectedDates);
              },
            },
          },
        },
      },
      series: [],
    };

    const xAxisValues = [];
    const profitSeries = { name: 'Profit', data: [] };
    const fbaSalesSeries = { name: 'FBA Sales', data: [] };
    const fbmSalesSeries = { name: 'FBM Sales', data: [] };

    this.chartData.item.forEach((currentItem) => {
      xAxisValues.push(currentItem.date);
      profitSeries.data.push(currentItem.profit);
      fbaSalesSeries.data.push(currentItem.fbaAmount);
      fbmSalesSeries.data.push(currentItem.fbmAmount);
    });

    chartOptions.xAxis.categories = [...xAxisValues];
    chartOptions.series = [profitSeries, fbaSalesSeries, fbmSalesSeries];

    Highcharts.chart('container', chartOptions);
  },
};
</script>

<style></style>
