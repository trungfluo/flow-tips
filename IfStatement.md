This short article is just a resume of a recent Node.js talk and two other older articles:

- [Effective Typed JavaScript](https://www.youtube.com/watch?v=jr5UYW6N2j8)
- [Modeling business problems with ADTs in Flow/js](http://www.adamsolove.com/js/flow/type/2016/04/13/modeling-with-adts.html)
- [Slaying a UI Antipattern with Flow](https://medium.com/@gcanti/slaying-a-ui-antipattern-with-flow-5eed0cfb627b)

One of the most common problems that developers have to face is the evolution of business model which requires partial or total code refactoring. 

These three resources above took a same approach to solve the problem by using [Disjoint Unions](https://flow.org/blog/2015/07/03/Disjoint-Unions/) and [Type Refinements](https://flow.org/en/docs/lang/refinements/). These two practices keep our code follow these two rules (first defined by [Yaron Minsky](https://blog.janestreet.com/author/yminsky/) in [Effective ML](https://blog.janestreet.com/effective-ml/)):

**- Make illegal states unrepresentable**

**- Code for exhaustiveness**

In the past our team use union without having a specific property or attribute which allows us to distinguish business cases. As our business model evolved, we have had serious problem to redefine the model even completely rewrite it. I think that it is fairly enough to make a note here to recommend these pratices to everyone who wants to get started with Flow and make good use its [Union type](https://flow.org/en/docs/types/unions/).
