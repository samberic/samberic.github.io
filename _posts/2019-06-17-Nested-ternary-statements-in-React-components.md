---
layout:     post
title:      Nested ternary statements in React components
date:       2019-06-17 20:24:52
summary:
categories:
---
After discovering our ESLinter hasn't been running for *some time* I've spent most of today going through trying to fix a whole bunch of eslint and a11y issues in our React app.  I hit quite an interesting one:
````
/*eslint no-nested-ternary: "error"*/
````

Which basically means, don't do this:
````
const drink = dislikeCoke ? 'fanta' : likesCherry ? 'cherryCoke' : 'dietCoke';
````
Which I think in general everyone can get behind as a rule.  It's not readable code, even when split over multiple lines with indentations, and should be broken out into if statements.

However, this is a very common pattern in React where we can use ternary statements to do conditional rendering in a component.

````
 <h1>Data Loader!</h1>
        { this.state.loading ? 
        <h2>It is Loading.</h2>
          : this.state.data ? 
          <h2>{this.state.data}</h2>
          :<h2>There was no result!</h2> 
        }
````
        

(This is a very contrived example).

I poked around on the internet for a while and the best alternative I've found to this is to extract the second part of the ternary into a stateless functional component.  The component can still live in the same file so it's still quick and easy to comprehend, and I found it to be a nice way to encapsulate the UI code.

````
const DataDisplay = ({data}) => data ? 
          <h2>{data}</h2>
          :<h2>There was no result!</h2> 
          
          ...
          
          { this.state.loading ? 
        <h2>It is Loading.</h2>
          : <DataDisplay data={this.state.data}/>
        }
````

Full example codepen below:
<p class="codepen" data-height="560" data-theme-id="0" data-default-tab="js,result" data-user="samberic" data-slug-hash="JQKbMd" style="height: 560px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;" data-pen-title="Improved Ternary React">
  <span>See the Pen <a href="https://codepen.io/samberic/pen/JQKbMd/">
  Improved Ternary React</a> by Sam Atkinson (<a href="https://codepen.io/samberic">@samberic</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://static.codepen.io/assets/embed/ei.js"></script>	