---
layout: post
current: post
cover: assets/images/hooks.jpg
navigation: True
date: 2020-07-30
tags: react
class: post-template
subclass: 'post'
title: Thinking in React Hooks
author: davidpfahler
description: >
  Updated example from the React documentation using modern React, i.e., functional components and hooks.
summary: >
  "Thinking in React" is an important beginner article in the React documentation and contains lessons even professionals should revisit from time to time. However, it has not been updated in some time and still features class based components. Using functional components and hooks, the tutorial becomes not only more up-to-date, but is also a lot cleaner.
---

["Thinking in React"](https://reactjs.org/docs/thinking-in-react.html) is a corner stone of the React documentation. This post shows how to implement the example using modern React, i.e. [functional components](https://reactjs.org/docs/components-and-props.html) and [hooks](https://reactjs.org/docs/hooks-intro.html). 

The original "Thinking in React" tutorial walks you through the thought process of building a searchable product data table using React. Here, I won't repeat this tutorial, but highlight the differences that show when using functional components and hooks. You can find my [full functional components and hooks version on GitHub](https://github.com/davidpfahler/thinking-in-react-hooks/blob/d07ba2f41e808d5220ede8184ec11a5a299d37a8/src/index.js).

> "Why didn't you submit a pull request to improve the React docs?" you might ask. Good question! I am in the process of finalizing a pull request and will update this post once it is live.

Quick recap: We have a mockup of a filterable product list and the product data as JSON:

![Mock up](assets/images/thinking-in-react-mock.png)

```json
[
  {category: "Sporting Goods", price: "$49.99", stocked: true, name: "Football"},
  {category: "Sporting Goods", price: "$9.99", stocked: true, name: "Baseball"},
  {category: "Sporting Goods", price: "$29.99", stocked: false, name: "Basketball"},
  {category: "Electronics", price: "$99.99", stocked: true, name: "iPod Touch"},
  {category: "Electronics", price: "$399.99", stocked: false, name: "iPhone 5"},
  {category: "Electronics", price: "$199.99", stocked: true, name: "Nexus 7"}
];
```

We have broken up the UI into components that ideally do one thing and one thing only. We then ordered these components in a hierarchy:

![Mock up with outlined components](assets/images/thinking-in-react-components.png)

- FilterableProductTable
  - SearchBar
  - ProductTable
    - ProductCategoryRow
    - ProductRow

Now lets compare these components from the original article to the components built with functional components and react.

## The Components

You can find the [full example code from the original "Thinking in React" article on CodePen](https://codepen.io/gaearon/pen/LzWZvb). You can find my [full functional components and hooks version on GitHub](https://github.com/davidpfahler/thinking-in-react-hooks/blob/d07ba2f41e808d5220ede8184ec11a5a299d37a8/src/index.js).

### FilterableProductTable

Original code from "Thinking in React":
```javascript
class FilterableProductTable extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      filterText: '',
      inStockOnly: false
    };
    
    this.handleFilterTextChange = this.handleFilterTextChange.bind(this);
    this.handleInStockChange = this.handleInStockChange.bind(this);
  }

  handleFilterTextChange(filterText) {
    this.setState({
      filterText: filterText
    });
  }
  
  handleInStockChange(inStockOnly) {
    this.setState({
      inStockOnly: inStockOnly
    })
  }

  render() {
    return (
      <div>
        <SearchBar
          filterText={this.state.filterText}
          inStockOnly={this.state.inStockOnly}
          onFilterTextChange={this.handleFilterTextChange}
          onInStockChange={this.handleInStockChange}
        />
        <ProductTable
          products={this.props.products}
          filterText={this.state.filterText}
          inStockOnly={this.state.inStockOnly}
        />
      </div>
    );
  }
}
```

With functional components and hooks:
```javascript
const FilterableProductTable = ({products}) => {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);
  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        setFilterText={setFilterText}
        setInStockOnly={setInStockOnly}
      />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly}
      />
    </div>
  );
}
```

It is actually shocking how much of a difference using hooks and functional components makes in terms of overall code quality.

Below, please find the rest of the components compared:

### SearchBar

Original:
```javascript
class SearchBar extends React.Component {
  constructor(props) {
    super(props);
    this.handleFilterTextChange = this.handleFilterTextChange.bind(this);
    this.handleInStockChange = this.handleInStockChange.bind(this);
  }
  
  handleFilterTextChange(e) {
    this.props.onFilterTextChange(e.target.value);
  }
  
  handleInStockChange(e) {
    this.props.onInStockChange(e.target.checked);
  }
  
  render() {
    return (
      <form>
        <input
          type="text"
          placeholder="Search..."
          value={this.props.filterText}
          onChange={this.handleFilterTextChange}
        />
        <p>
          <input
            type="checkbox"
            checked={this.props.inStockOnly}
            onChange={this.handleInStockChange}
          />
          {' '}
          Only show products in stock
        </p>
      </form>
    );
  }
}
```

With functional components and hooks:
```javascript
const SearchBar = ({filterText, inStockOnly, setFilterText, setInStockOnly}) => (
  <form>
    <input
      type="text"
      placeholder="Search..."
      value={filterText}
      onChange={(e) => setFilterText(e.target.value)}
    />
    <p>
      <label>
        <input
          type="checkbox"
          checked={inStockOnly}
          onChange={(e) => setInStockOnly(e.target.checked)}
        />
        {' '}
        Only show products in stock
      </label>
    </p>
  </form>
);
```

### ProductTable

Original:
```javascript
class ProductTable extends React.Component {
  render() {
    const filterText = this.props.filterText;
    const inStockOnly = this.props.inStockOnly;

    const rows = [];
    let lastCategory = null;

    this.props.products.forEach((product) => {
      if (product.name.indexOf(filterText) === -1) {
        return;
      }
      if (inStockOnly && !product.stocked) {
        return;
      }
      if (product.category !== lastCategory) {
        rows.push(
          <ProductCategoryRow
            category={product.category}
            key={product.category} />
        );
      }
      rows.push(
        <ProductRow
          product={product}
          key={product.name}
        />
      );
      lastCategory = product.category;
    });

    return (
      <table>
        <thead>
          <tr>
            <th>Name</th>
            <th>Price</th>
          </tr>
        </thead>
        <tbody>{rows}</tbody>
      </table>
    );
  }
}
```

With functional components and hooks:
```javascript
const ProductTable = ({products, filterText, inStockOnly}) => {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.name.toLowerCase().indexOf(filterText.toLowerCase()) === -1) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}
```

### ProductCategoryRow

Original:
```javascript
class ProductCategoryRow extends React.Component {
  render() {
    const category = this.props.category;
    return (
      <tr>
        <th colSpan="2">
          {category}
        </th>
      </tr>
    );
  }
}
```

With functional components and hooks:
```javascript
const ProductCategoryRow = ({category}) => (
  <tr>
    <th colSpan="2">
      {category}
    </th>
  </tr>
);
```

### ProductRow

Original:
```javascript
class ProductRow extends React.Component {
  render() {
    const product = this.props.product;
    const name = product.stocked ?
      product.name :
      <span style={{color: 'red'}}>
        {product.name}
      </span>;

    return (
      <tr>
        <td>{name}</td>
        <td>{product.price}</td>
      </tr>
    );
  }
}
```

With functional components and hooks:
```javascript
const ProductRow = ({product}) =>  {
  const color = product.stocked ? 'inherit' : 'red';

  return (
    <tr>
      <td><span style={{color}}>{product.name}</span></td>
      <td>{product.price}</td>
    </tr>
  );
}
```