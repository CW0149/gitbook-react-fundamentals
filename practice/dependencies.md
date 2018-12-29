# 常用的React包

### [react-select](https://github.com/JedWatson/react-select)
### [react-virtualized](https://github.com/bvaughn/react-virtualized)
### 示例
#### react-select与react-virtualized搭配实现下拉大量数据平滑滚动
[点击搜索icon查看效果](https://notebook.huolong.tk/github-rate)

```
/** Select.js **/
const components = {
  Control,
  MenuList,
  MultiValue,
  NoOptionsMessage,
  Placeholder,
  SingleValue,
  ValueContainer,
}
function MenuList(props) {
  return (
    <VList list={props.children} width={200} />
  )
}
<Select
  classes={classes}
  styles={selectStyles}
  options={suggestions}
  components={components}
  value={this.state.single}
  onChange={this.handleChange('single')}
  placeholder="Search Languages"
/>
/** VList.js **/
import React from 'react'
import PropTypes from 'prop-types'
import List from 'react-virtualized/dist/commonjs/List'
function rowRenderer(props) {
  return ({
    key,         // Unique key within array of rows
    index,       // Index of row within collection
    isScrolling, // The List is currently being scrolled
    isVisible,   // This row is visible within the List (eg it is not an overscanned row)
    style        // Style object to be applied to row (to position it)
  }) => {
    return (
      <div
        key={key}
        style={style}
        className="vlist-option"
      >
        {props.list[index]}
      </div>
    )
  }
}
class VList extends React.PureComponent {
  render() {
    const { props } = this
    return (
      <List
        style={{padding: '10px 0 0', height: 'auto !important', maxHeight: '300px', outline: 'none'}}
        width={props.width}
        height={300}
        rowCount={props.list.length}
        rowHeight={40}
        rowRenderer={rowRenderer(props)}
        noRowsRenderer={() => (
          <div style={{paddingBottom: '10px'}}>No Options</div>
        )}
      />
    )
  }
}
VList.propTypes = {
  list: PropTypes.array.isRequired
}
export default VList
```