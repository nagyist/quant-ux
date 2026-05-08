
<template>
    <div class="MatcWidgetTypeSVGIcon" >
        <div class="MatcWidgetTypeSVGIconWrapper" ref="wrapper">
            <svg 
                xmlns="http://www.w3.org/2000/svg" 
                :class="'MatcWidgetTypeSVGIconSVG'"
                :width="width" 
                :height="height" 
                :viewBox="viewBox" 
                stroke-width="1.5" 
                stroke="currentColor" 
                fill="none" 
                stroke-linecap="round" 
                stroke-linejoin="round"
                v-html="path"
                ref="svg"
                >
            </svg>
        </div>
    </div>
  </template>
  <script>
  import DojoWidget from "dojo/DojoWidget";
  import UIWidget from "core/widgets/UIWidget";

  export default {
    name: "SVGIcon",
    mixins: [UIWidget, DojoWidget],
    data: function() {
      return {
        width: 40,
        height: 40,
        path: '',
        viewBox: '0 0 24 24',
        loadedFileURL: null,
        placeholderPath: `
          <line x1="1" y1="1" x2="23" y2="23" stroke="currentColor" />
          <line x1="23" y1="1" x2="1" y2="23" stroke="currentColor" />`
      };
    },
    components: {},
    computed: {
    },
    methods: {
      postCreate () {
        this._borderNodes = [];
        this._backgroundNodes = [];
        this._shadowNodes = [];
      },
  
      wireEvents () {
        this.own(this.addClickListener(this.domNode, e => {
          this.emitClick(e)
        }));
        this.wireHover()
      },
  
      resize (box) {
        this.width = box.w
        this.height = box.h
      },
  
      render (model, style, scaleX) {
        this.style = style
        this.model = model
        this.scale = scaleX
        this._renderIconMarkup(model)
        this._applyStyle(style, scaleX)
        if (style.backgroundImageRotation !== undefined) {
            this.$refs.wrapper.style.transform  = `rotate(${style.backgroundImageRotation}deg)`
        }
        this.resize(model)
      },

      setAnimatedStyle (style) {
        this._applyStyle(style, this.scale)
      },    

      async _renderIconMarkup (model) {
        const image = this._getPrimaryImage(model)
        if (image) {
          await this._loadFromFile(image)
          return
        }

        const svg = model && model.props ? model.props.svg : ''
        this._setMarkup(svg)
      },

      _getPrimaryImage (model) {
        console.log('SVGIcon._getPrimaryImage()', model)
        const props = model && model.props ? model.props : null
        if (!props) {
          return null
        }

        if (props.images && props.images.length > 0) {
          return props.images[0]
        }

        return null
      },

      _applyStyle (style, scaleX = 1) {
        if (!this.$refs.svg || !style) {
          return
        }
        const color = style.color || (this.model && this.model.props ? this.model.props.color : null) || '#333333'
        this.$refs.svg.style.color = color
        this.$refs.svg.style.strokeWidth = (style.strokeWidth || 1) * scaleX
      },

      _setMarkup (svgText) {
        this.loadedFileURL = null
        if (!svgText) {
          this.path = this.placeholderPath
          this.viewBox = '0 0 24 24'
          return
        }

        if (svgText.indexOf('<svg') >= 0) {
          this._setMarkupFromSVG(svgText)
          return
        }

        this.path = svgText
        this.viewBox = '0 0 24 24'
      },

      _setMarkupFromSVG (svgText) {
        const parser = new DOMParser()
        const documentNode = parser.parseFromString(svgText, 'image/svg+xml')
        const svgNode = documentNode.querySelector('svg')
        if (!svgNode) {
          this.path = this.placeholderPath
          this.viewBox = '0 0 24 24'
          return
        }

        this._normalizeStroke(svgNode)
        this.path = svgNode.innerHTML

        const box = svgNode.getAttribute('viewBox')
        if (box) {
          this.viewBox = box
        } else {
          this.viewBox = '0 0 24 24'
        }
      },

      _normalizeStroke (svgNode) {
        const allNodes = svgNode.querySelectorAll('*')
        allNodes.forEach(node => {
          const stroke = node.getAttribute('stroke')
          if (stroke && stroke !== 'none') {
            node.setAttribute('stroke', 'currentColor')
          }
        })
      },

      _getFileURL (file) {
        if (!file) {
          return null
        }

        if (typeof file === 'string') {
          if (file.indexOf('http://') === 0 || file.indexOf('https://') === 0 || file.indexOf('/') === 0 || file.indexOf('data:') === 0) {
            return file
          }

          if (this.hash) {
            return `/rest/images/${this.hash}/${file}`
          }

          if (this.jwtToken) {
            return `/rest/images/${file}?token=${this.jwtToken}`
          }

          return `/rest/images/${file}`
        }

        if (!file.url) {
          return null
        }

        if (this.hash) {
          return `/rest/images/${this.hash}/${file.url}`
        }

        if (this.jwtToken) {
          return `/rest/images/${file.url}?token=${this.jwtToken}`
        }

        return `/rest/images/${file.url}`
      },

      async _loadFromFile (file) {
        const fileURL = this._getFileURL(file)
        if (!fileURL) {
          this.path = this.placeholderPath
          this.loadedFileURL = null
          return
        }

        if (this.loadedFileURL === fileURL) {
          return
        }

        try {
          const result = await fetch(fileURL)
          if (!result.ok) {
            this.path = this.placeholderPath
            this.loadedFileURL = null
            return
          }
          const svgText = await result.text()
          this._setMarkupFromSVG(svgText)
          this.loadedFileURL = fileURL
        } catch (err) {
          this.path = this.placeholderPath
          this.loadedFileURL = null
          this.logger.error('SVGIcon._loadFromFile()', 'Could not load SVG file', err)
        }
      },
  
      getValue () {},
  
      setValue() {},
  
      getState () {
        return {};
      },
  
      setState () {}
    },
    mounted() {
    }
  };
  </script>