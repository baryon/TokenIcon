<template>
    <svg ref="svg" viewBox="0 0 100 100"  preserveAspectRatio="none">
    </svg>
</template>
<script>
import * as d3 from 'd3'
const shapeCount = 3

export default {
  name: 'token-icon',
  props: {
    contractId: {
      type: String
    },
    background: {
      type: String,
      default: 'transparent'
    }
  },
  methods: {
    // get 6 HEX colors from address
    getColors (account) {
      const colors = []
      for (let i = 0; i < account.length; i += 6) {
        if (account.length - i !== account.length % 6) colors.push(account.slice(i, i + 6))
      }
      return colors
    },

    // count occurences of letters
    getLettersCount (account) {
      const letters = account.split('')
      const countItems = letters.reduce( (count, curr, i) => {
        (typeof count[curr] === 'undefined') ? count[curr] = [i] : count[curr].push(i)
        return count
      }, {})

      return countItems
    },

    // get coords from address
    getCoords (countLetters) {
      const lettersPos = {}
      d3.range(10)
        .map(d => '' + d)
        .concat('a', 'b', 'c', 'd', 'e', 'f')
        .forEach((d, i) => { lettersPos[d] = i })

      const tmpCoords = []
      Object.keys(countLetters).forEach( k => {
        countLetters[k].forEach(p => { tmpCoords.push([ p, lettersPos[k] ]) })
      })

      const maxX = d3.max(tmpCoords.map(d => d[0]))
      const maxY = d3.max(tmpCoords.map(d => d[1]))

      // map a number to a range
      function map (num, in_min, in_max, out_min, out_max) {
        return (num - in_min) * (out_max - out_min) / (in_max - in_min) + out_min
      }

      return tmpCoords.map(d => [
        map(d[0], 0, maxX, 10, 100 - 10),
        map(d[1], 0, maxY, 10, 100 - 10)
      ])
    },

    // use only most used characters
    // limit is the number of items to include
    getShapes (coords, countLetters, limit) {
      const mostUsed = Object.keys(countLetters)
        .filter(k => countLetters[k].length > 2)
      mostUsed.length = limit // limit array length

      // get shape coords
      return Object.keys(countLetters)
        .filter(l => mostUsed.includes(l) )
        .map(k => countLetters[k].map(i => ({ coords: coords[i], letter: k })))
    },
    // draw as SVG
    drawSVG (shapesCoords, colors) {
      // helper function
      const renderPath = (d) => {
        return d == null ? null : 'M' + d.join('L') + 'Z'
      }

      const svg = d3.select(this.$refs.svg)
        .style('background', this.background)

      svg.selectAll('*').remove()

      svg.append('g')
        .classed('shapes', true)
        .selectAll('path.shape')
        .data(shapesCoords)
        .enter()
        .append('path')
        .classed('shapes', true)
        .attr('d', d => renderPath(d.map(d => d.coords)))
        .style('fill', (d, i) => colors[i])
    }
  },
  mounted () {
    const colors = this.getColors(this.contractId)
    const letters = this.getLettersCount(this.contractId)
    const coords = this.getCoords(letters)
    const shapeCoords = this.getShapes(coords, letters, shapeCount)
    this.drawSVG(shapeCoords, colors)
  }
}
</script>

<style lang="sass">
svg
  margin: 25px
  path
    fill: none
    stroke: #76BF8A
    stroke-width: 3px
</style>
