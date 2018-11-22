---
layout: post
title:  "What is react"
categories: react
date:   2018-11-22 16:56:05 +0800
---
This question has been on my mind for some time. I saw React on Medium posts, heard it at meetings but never got down to answering the question.
So when I saw a free talk by General Assembly titled 'Why learn react', I reacted 'Finally, I can get some answers'.<br>
It was a great introduction. The instructor directed me to https://codepen.io/pen to try the code hands on.
There, i had to import react and react dom in the Javascript tab and choose babel as the Javascript preprocessor.<br><br>

**My takeaways** <br>
React is a Javascript library for building UI (what appears on a website).
A webpage can be composed of several components.
For example, each functionality on a webpage can be 1 component.
React allows us to build up a website by creating components that are reusable.


A component can be created in the Javascript tab as follows

{% highlight react %}
class Intro extends React.Component {

    // render method 
    render() {
        return (
           <h1>
           	Hello guys
          </h1>
        )
    }
}

{% endhighlight %}

An instance of the component class named Intro can be created by the command ``<Intro />``.
When created, it will return a jsx code (JavaScript eXtension; allows you to write html in javascript) which contains a header named 'Hello guys'.
The render method in a component can only return 1 element (div, h1, etc)

To output on a webpage, we have to modify the DOM by adding the following in the Javascript tab.
We see the instance of the class Intro created and returning output at the root div.
{% highlight react %}
ReactDOM.render(
	<Intro />,
	document.getElementById("root")
)
{% endhighlight %}

The html code contains			
{% highlight html %}
<div id="root"></div>
{% endhighlight %}

The output on the website will be **Hello guys**

The ReactDOM.render method above can only have 1 component class which is the parent class and 1 DOM element.
For multiple different class instances to be created, they will be created in a parent class. They will be subclasses.
For example
{% highlight react %}
class App extends React.Component {

    //  render method 
    render() {
        return (
          <div>
          <Intro />
          <h1>
           	Hello everyone
          </h1>
          </div>
        )
    }
}
{% endhighlight %}

Now App is the parent class that creates the instance of children class/subclass Intro (defined above).
We then change the ReactDom method to create the instance of parent class App
{% highlight react %}
ReactDOM.render(
	<App />,
	document.getElementById("root")
)
{% endhighlight %}

Now the output on the website will be **Hello guys** followed by **Hello everyone** on the next line. 

I can also create instance of Intro class multiple times by issuing multiple <Intro /> commands in the App parent class component.
This will print Hello guys multiple times.
<br><br>

*Passing data using Props*<br>
I can pass in parameters to a class component when i create an instance of it (see below: *country*, *state*).
The parameters in the class component are referred by ``{}`` braces containing this.props.*name*

{% highlight react %}
class App extends React.Component {

    //  render method 
    render() {
      var greet = " Konnichiwa"
        return (
          <div>
          <Intro />
          <Intro />
          <h1>
           	Hello everyone in {this.props.country}, {this.props.state}. {greet}
          </h1>
          </div>
        )
    }
}

ReactDOM.render(
	<App country="Japan" state="Tokyo"/>,
	document.getElementById("root")
)
{% endhighlight %}

This will print *Hello everyone in Japan, Tokyo. Konnichiwa*.<br><br>

*State*<br>
A component can have a state. A state is a component's internal value and only affects that component.
When the state changes, react will rerender the component (render method) and update the DOM if there are any changes.
In addition, only the DOM element that is changed will be updated. The rest will be unaffected.
This allows the webpage to react quickly to changes. Search virtual DOM.
See [state process][state-process]

To initialize a state. we can include it in the constructor method.
This will be called when the class component is created. 
In following code, the state name is initialized to *``jack's``*.
We then create a button titled *``i am <state name> ``* which outputs *``i am jack's ``*.
When it is clicked, the method reveal will be called.
The method reveal will call setState that will update the state name to *``Jack's brother``*.
Thus when the state is updated, react will rerender and make the change to the button and display *``i am jack's brother``*

{% highlight react %}
class App extends React.Component {
  constructor(){
    super();
    this.state={name:"jack's "};
    this.reveal = this.reveal.bind( this );
    }
  
  reveal(){
    this.setState({name:this.state.name + " brother"}) 
  }
    //  render method 
    render() {
      var greet = " Konnichiwa"
        return (
          <div>
          <Intro />
          <Intro />
          <h1>
           	Hello everyone in {this.props.country}, {this.props.state}. {greet}
          </h1>
          <button onClick={this.reveal}> i am {this.state.name} </button>
          </div>
        )
    }
}
{% endhighlight %}
In the constructor, we have to bind the method reveal to **this** so the method will be called 
on the current instance. Refer to this [bind video][bind-video].

I am glad I heard about the basics of React. After class, I proceeded to try a Tic tac toe example which I will talk about the next post.
Other things to explore are the React build tool, building dashboards and Vue js.

[state-process]: https://www.youtube.com/watch?v=qh3dYM6Keuw&index=2&list=PLwAyfvNbx0t_7hsHJSkAQIS1uD6BqNpD4&t=0s
[bind-video]: https://www.youtube.com/watch?v=g2WcckBB_q0&list=PLwAyfvNbx0t_7hsHJSkAQIS1uD6BqNpD4&index=2