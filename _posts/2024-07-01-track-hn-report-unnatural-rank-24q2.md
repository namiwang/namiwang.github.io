---
layout: single
title: 'Track HN: Stories with Unnatural Rank Changing on Hacker News April-June 2024'
date: '2024-05-10 19:00:00 +0800'
classes: wide
---

## About this Report

This post is part of the [Track HN](https://www.track-hacker-news.com) project. It tracks the activity of stories on Hacker News that show unnatural behavior.

### Criteria

- submitted between April 1, 2024 and July 1, 2024
- more than 

### Data Source

The data is collected from the [Hacker News API](TBD)

## Stories with Unnatural Activity

{% for s in site.data.thn.2407.report.stories %}
  <h3>
    <a href="https://dash.track-hacker-news.com/stories/{{ s.id }}">
      {{ s.id }} - {{ s.title }}
    </a>
  </h3>

  <!-- TODO responsive -->
  <!-- https://echarts.apache.org/handbook/en/concepts/chart-size/ -->
  <div style="width:100%; min-width:480px; height:400px;">
    <div id="story-{{ s.id }}" style="width:100%; height:100%;"></div>
  </div>
{% endfor %}

<!--  -->

<script src="https://cdn.jsdelivr.net/npm/echarts@5.5.0/dist/echarts.min.js"></script>

<script>
  const renderChart = async (story) => {
    const element = document.getElementById(`story-${story.id}`)
    story = await (await fetch(`/assets/data/thn/report-24q2/${story.id}.json`)).json()

    const chart = echarts.init(element)
    const option = {
      dataset: [
        // 0 - scores
        {
          dimensions: ['tracked_at', 'score'],
          source: story.scorestamps,
        },
        // 1 - scores, sorted
        {
          fromDatasetIndex: 0,
          transform: {
            type: 'sort',
            config: { dimension: 'tracked_at', order: 'asc' }
          },
        },

        // 2 - comments count
        {
          dimensions: ['tracked_at', 'count'],
          source: story.descendants_count_stamps,
        },
        // 3 - comments count, sorted
        {
          fromDatasetIndex: 2,
          transform: {
            type: 'sort',
            config: { dimension: 'tracked_at', order: 'asc' }
          },
        },

        // 4 - rank
        {
          dimensions: ['tracked_at', 'rank'],
          source: story.rankstamps,
        },
        // 5 - rank, sorted
        {
          fromDatasetIndex: 4,
          transform: {
            type: 'sort',
            config: { dimension: 'tracked_at', order: 'asc' }
          },
        },
      ],
      // title: {
      //   text: ''
      // },
      tooltip: {
        trigger: 'axis'
      },
      legend: {
        data: ['score', 'comments count', 'rank']
      },
      grid: {
        left: '3%',
        right: '3%',
        bottom: '3%',
        containLabel: true
      },
      toolbox: {
        feature: {
          saveAsImage: {}
        }
      },
      xAxis: {
        type: 'time',
      },
      yAxis: [
        {
          type: 'value'
        },
        {
          type: 'value',
          splitLine: {
            show: false
          },
          inverse: true,
        }
      ],
      series: [
        {
          name: 'score',
          type: 'line',
          datasetIndex: 1,
        },
        {
          name: 'comments count',
          type: 'line',
          stack: 'Total',
          datasetIndex: 3,
        },
        {
          name: 'rank',
          type: 'line',
          datasetIndex: 5,
          yAxisIndex: 1
        }
      ]
    }
    chart.setOption(option)

    // responsive chart size
    // https://echarts.apache.org/handbook/en/concepts/chart-size#reactive-of-the-container-size
    window.addEventListener('resize', function() {
      chart.resize()
    })
  }

  let stories = {{ site.data.thn.2407.report.stories | jsonify }}
  console.log(stories);

  for (let story of stories) {
    renderChart(story)
  }
</script>
