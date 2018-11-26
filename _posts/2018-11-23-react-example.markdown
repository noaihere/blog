---
layout: post
title:  "Connect four with react"
categories: react example
date:   2018-11-23 16:55:05 +0800
---
There is a Tic-tac-toe React tutorial at [react tic-tac][react-tic-tac].
However I wanted to see how connect four can be implemented using React.
The code I have referred to is at [link][link] .

The first step is to create the 6 x 7 connect four grid.
First a component class called App is created with a constructor that initializes a state **board** as a empty array ``[]``. <br>
{% highlight react %}
  constructor(props){
    super(props)
    this.state={board:[]}
	this.play = this.play.bind(this);
}

{% endhighlight %}
Next a method initBoard will create the grid using 2 for loops that will populate the 2 dimensional
array board with nulls. It will also have a setState method so that the state board will be updated.
To access the 1st row and 3rd column we can issue the command **board[0][2]**.<br>

{% highlight react %}
 initBoard() {
    // Create a blank 6x7 matrix
    let board = [];
    for (let r = 0; r < 6; r++) {
      let row = [];
      for (let c = 0; c < 7; c++) { row.push(null) }
      board.push(row);
    }
   this.setState({
     board:board
   })
 }
{% endhighlight %}


A method componentWillMount will call the method initBoard and this will happen before the initial render.
See [pre mounting][pre-mounting]
{% highlight react %}
    componentWillMount() {
    this.initBoard();
  }
{% endhighlight %}



We can then add the render method that will return the actual grid by mapping through
each element in the 2d array and calling the functions Row and Cell so a 6 x 7 grid of boxes is outputted.
The function Row  will create a <tr> and call the function Cell that will create <td> in a html table.
The use of functions and their syntax is explained here [functions][functions].
We also note that in the map function, the first argument refers to an element in the array and the second argument, the index of that element.
A div with the classname *cell* will be contained within each <td> to produce each square (as defined in CSS).
Another div with classname *white* will be contained within the square div to produce the empty circles in the squares.
{% highlight react %}
   render() {
      console.log(this.state.board)
        return (
        <div>
        <div className="button">New Game</div>
       <table>
          <tbody>
            {this.state.board.map((row, i) => (<Row  row={row} play={this.play}  />))}
          </tbody>
        </table>
      </div>
        )
    }
}

const Row = ({ row , play}) => {
  return (
    <tr>
      {row.map((cell, i) => <Cell value={cell} columnIndex={i} play={play} />)}
    </tr>
  );
};

const Cell = ({ value, columnIndex , play}) => {
  let color = 'white';
  if (value === 1) {
    color = 'red';
  } else if (value === 2) {
    color = 'yellow';
  }
  return (
    <td >
      <div className="cell" onClick={()=>{play(columnIndex)}}>
           <div className={color}></div>
        {columnIndex}
      </div>
    </td>
  );
};

{% endhighlight %}
Now we need to add the functionality that when we drop a circle, it will go to the bottom of the grid.
When we click anywhere within a particular column, the circle will go to the bottom of that column.
Thus we need to add an onclick function *play* in the div of the square taking the column index as the parameter.
And we have to pass this play function from the map function in the render method to the Row and Cell functions as a parameter.
(See above code snippet)

The play function will loop through each row of a particular column and check if the bottom row of
the particular column is filled. If it is not filled, it will assign a value 1 or 2 (depending on which player's turn) to that element of the 2D array **board**. 
And break out of the loop. Otherwise, it will go to the 2nd bottom-est row and do the same. 
The setState method will then be called on the board.
We have to also bind this play method in the constructor. (See first code snippet on top)

{% highlight react %}
  play(c){
    let board= this.state.board
        for (let r = 5; r >= 0; r--) {
        if (!board[r][c]) {
          board[r][c] = 1;
          break;
        }
      }
    this.setState(board)
  }
{% endhighlight %}

Now we need to define the current player's turn. So that the colour of the circle dropped will differ depending on whose turn is it.
We define in the constructor state player1 to be 1, and state player2 to be 2. And state currentPlayer to be null.
And in the initBoard method, we add to the setState method, the currentPlayer to be this.state.player1.
To switch player after the first player has completed his turn, we add to the setState method in the play method,
``currentPlayer: this.togglePlayer()``. This will set the state currentPlayer to be the output of the togglePlayer method.
The method togglePlayer will check if currentPlayer state = player1 and if yes, make currentPlayer
state equals to player2.
{% highlight react %}
    togglePlayer() {
    return (this.state.currentPlayer === this.state.player1) ? this.state.player2 : this.state.player1;
  }
{% endhighlight %}

Next we need to determine if any player has won.
There are 4 ways the code decides is a winning combination.
Horizontal, vertical, diagonal right and diagonal left.
The four methods return the value of one of the elements in the winning combination (4 in a row).
A method checkAll will combine the output of those 4 methods in a or statement.
This method checkAll will then be called in the play method after a player makes a move and check if it is equal to state of player1, 2 or 'draw'.
If any of the 3 conditions are true, the setState will then be called and update the state gameOver and message.
This will end the game. Otherwise it will update the state Board, and currentPlayer by calling the togglePlayer method. Then the game continues.<br>

I have explained how the code in [link][link] implements connect four in React. There are some additional features that could be implemented that is seen in
[react tic-tac][react-tic-tac] like maintaining history. I would like to explore them next time.



[react-tic-tac]: https://reactjs.org/tutorial/tutorial.html
[link]: https://codepen.io/jeffleu/pen/KgbZwj
[pre-mounting]: https://developmentarc.gitbooks.io/react-indepth/content/life_cycle/birth/premounting_with_componentwillmount.html
[functions]: https://javascriptplayground.com/functional-stateless-components-react/ 
