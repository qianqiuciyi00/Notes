```
// 雷达图 自定义name内容及边框
radar: [{
    name: {
    formatter: (value) => {
        return value == this.ladarTitle ? `{active|${value}}  {activeArrow|>}` : `{common|${value}}  {arrow|>}`
    },
    textStyle: {
        rich: {
        active: {
            fontFamily: 'SourceHanFont',
            fontSize: '18',
            fontWeight: '400',
            color: '#FFFFFF',
            padding: [8, 8, 8, 11],
            borderWidth: 1,
            borderRadius: 4,
            borderColor: '#0551a5',
            backgroundColor: '#081638',
            shadowColor: '#0551a5',
            shadowBlur: 26
        },
        common: {
            fontFamily: 'SourceHanFont',
            fontSize: '18',
            fontWeight: '400',
            color: '#FFFFFF',
            borderWidth: 1,
            borderRadius: 4,
            padding: [8, 8, 8, 11],
            borderColor: 'rgba(255, 255, 255, 0.25)'
        },
        arrow: {
            height: 12,
            align: 'center',
            padding: [0, 0, 0, 2],
            backgroundColor: {
            image: './static/images/arrow.png'
            }
        },
        activeArrow: {
            height: 14,
            align: 'center',
            padding: [0, 0, 0, 2],
            backgroundColor: {
            image: './static/images/arrow-active.png'
            }
        },
        }
    }
    }
}]
```