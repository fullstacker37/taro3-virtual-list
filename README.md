# 基于Taro3的虚拟列表

## 使用方法
```
import { TaroVirtualList } from 'taro-virtual-list'

export default function Demo(): JSX.Element {
  // 模拟list数据
  const [list, setList] = useState<number[]>([])

  // 设置list
  useEffect(() => {
    const arr: number[] = []
    Array(84).fill(0).forEach((item, index) => {
      arr.push(index)
    })
    setList(arr)
  }, [])
  // 渲染列表Item
  const renderFunc = (item, index, pageIndex) => {
    return (
      <View className="el">{`当前是第${item}个元素，是第${pageIndex}屏的数据`}</View>
    )
  }
  const handleBottom = () => {
    console.log('触底了')
  }
  const handleComplete = () => {
    console.log('加载完成')
  }
  return (
    <View>
      <TaroVirtualList
        list={list}
        onRender={renderFunc}
        onBottom={handleBottom}
        onComplete={handleComplete}
        scrollViewProps={{
          style: {
            "height": '100vh',
          },
        }}
      />
    </View>
  )

}
```

## 为啥要开发该组件
1. 列表页数据量过多，一次性渲染完成后页面节点数量过大，造成页面渲染卡顿，渲染完成之后操作页面数据也会异常卡顿；
2. 官方虚拟列表（3.2.1）存在一定的渲染bug，特别是针对**列表节点不等高**，存在诸多问题，比如节点闪动、滚动过快造成无限加载、白屏率较高等；

## 该组件适用场景
1. 页面节点渲染较多（主要是列表页）；
2. 针对列表页**节点不等高**具有更好的支持；

## 参数说明

### Props
| 参数 | 类型 | 默认值 | 必填 | 说明 |
| --- | :----: | ---- | ---- | ------ |
| list | Array | - | 是 | 列表数据 |
| listId | String | "zt-virtial-list" | 否 | 虚拟列表唯一id（防止同一个页面有多个虚拟列表导致渲染错乱）|
| segmentNum | Number | 10 | 否 | 自定义二维数组每一维度的数量，默认10 |
| autoScrollTop | Boolean | true | 否 | 组件内部是否需要根据list数据变化自动滚动至列表顶部 |
| screenNum | Number | 2 | 否 | 指定监听页面显示区域基准值，例如2，则组件会监听 2 * scrollHeight高度的上下区域范围(该值会影响页面真实节点的渲染数量，值越大，白屏几率越小，但是页面性能也就越差) |
| scrollViewProps | Object | - | 否 | 自定义scrollView的参数，会合并到组件内部的scrollView的参数里 |

### Events
| 参数 | 回调参数 | 默认值 | 必填 | 说明 |
| --- | :----: | ---- | ---- | ------ |
| onRender | (item, index, segmentIndex) => {}<br>item: 列表的单个数据项的值;<br> index：列表的单个数据项的index;<br>segmentIndex：当前二维数组维度的index | - | 列表的渲染回调，用于自定义列表Item | - | 列表的渲染回调，用于自定义列表Item |
| onBottom | - | - | 否 | 列表是否已经触底回调 |
| onComplete | - | - | 否 | 列表是否已经把全部数据加载完成的回调 |
| onRenderTop | - | - | 否 | 列表上部分内容渲染回调，用于渲染插入虚拟列表上边的内容 |
| onRenderBottom | - | - | 否 | 列表下部分内容渲染回调，用于渲染插入虚拟列表下边的内容 |

## 注意事项
1. onRenderBottom渲染的内容会在虚拟列表所有数据渲染完成之后才会调用
2. 设置scrollViewProps参数的时候请注意：
  - 最好给个容器高度
  - 如果想触发onScrollToLower方法，可以尝试使用onBottom回调代替（因为组件内部已经使用了onScrollToLower方法，如果外部再定义，会导致代码冲突，组件上拉加载失效）

## 原理
1. 处理数据：将传入的list分割成二维数组，初始化的时候只加载二维数组的第一项，随着页面滚动，会依次加入对应维度的数据（加快了初始化渲染速度）；
2. 可视区域监听：利用Taro.createIntersectionObserver监听当前可视区域，将不在可视区域内的那一维度的数据改成该维度数据所占的视图高度（进而减少了setState的数据量）；
3. 渲染：将不在可视区域内的数据利用一个节点占高（减少了节点渲染数量）；
