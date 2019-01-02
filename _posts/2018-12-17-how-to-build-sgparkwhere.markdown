---
layout: post
title:  "How to build sgparkwhere"
date:   2018-12-17 17:54:05 +0800
categories: API
---

Head over to this [app link][app-link] to see the application. Wait for initial load.

This post will be on how the application was built. <br><br>

**1) Get data from LTA DataMall API**<br><br>
We refer to this [link][link] which is the LTA DataMall API User Guide.
On page 9 of the guide, there is a sample python 2.7 code  to retrieve the data.
Our code is based on this. <br>

*urlparse*<br>
urlparse is a method of the module urllib.parse.
It returns an object which acts like a tuple of 6 elements.
This class is called namedtuple and it differs from the normal tuple in that the fields can be accessed by name
(obj.attr) instead of just positional index. See [namedtuple link][namedtuple-link], [namedtuple link2][namedtuple-link2]
It has a geturl method that returns the real URL in case of redirection. See [geturl link][geturl-link]. 
To see the methods which the object *target* has, we issue ``dir(target)``.

*API request*<br>
It makes use of the python module httplib2.
The command http.Http() creates a HTTP client.
We issue a HTTP request with the request() method.
The parameters are the url, request type, body and headers.
The headers contain the AccountKey which is provided when we register for LTA DataMall.
The request() method returns a tuple of response and content.

*Json*<br>
The API returns JSON data with type bytes. We want to make it into a python object so that we can 
manipulate the data easily. To do that we use the json.loads method and assign it to jsonObj.
The type of jsonObj is a dictionary. See this useful [json video][json-vid]. 
The user guide outputs the json as a string to a file using the method json.dump but we will not as we will work with the dictionary.<br><br>

**2) Transform data**<br>

Now that we have the data in python, we can transform it to the form we want.<br>

*pandas*<br>
Pandas is a python library for data manipulation and analysis.<br>
We transform the json dictionary to a pandas dataframe. <br>
We first access the value of the key = 'value' in the dictionary and it returns a list of dictionaries:<br>
``type(jsonObj['value'])`` *list*<br> 
``type(jsonObj['value'][0])`` *dict*<br><br> 
Make it into a pandas dataframe: <br>``df=pd.DataFrame.from_records(jsonObj['value'])`` or ``df=pd.DataFrame(jsonObj['value'])``<br>
Print out unique values of column Area: ``df['Area'].unique()`` <br>
Print out count of each unique value in column Area: ``df['Area'].value_counts()``<br>
Replace blank values with value 'others': ``df['Area'] = df['Area'].replace('', 'Others')``<br>

*accessing values*<br>
Return 4th row of all columns as pandas series : ``df.iloc[4]``<br>
Return 4th row of all columns as pandas dataframe : ``df.iloc[[4]]``<br>
Return first 5 rows of 5th column as pandas series : ``df.iloc[:5,5]``<br>
Return first 5 rows of 5th column as pandas dataframe : ``df.iloc[:5,[5]]``<br>
Return first row value in column Location: ``df['Location'].iloc[0]``<br>
Return first four rows in column Location as pandas series: ``df['Location'].iloc[:5]``<br>
See [pandas link][pandas-link]<br><br>

Split location column into 2 columns lat and long: <br>
``df['lat']=df['Location'].str.split(' ', expand=True)[0]``<br>
``df['long']=df['Location'].str.split(' ', expand=True)[1]``<br>

Get types of all columns in dataframe: ``df.dtypes``<br>
Get type of column lat in dataframe: ``df['lat'].dtype`` *'O' means string* <br>
Make column lat numeric: ``df['lat']= pd.to_numeric(df['lat'])``<br>
Get max lat: ``df['lat'].max()``<br>
Create new column dist and populate with value 0: ``df['dist'] = 0``<br><br>

**3) Visualising data**<br><br>
Now that we have the transformed data in python, we can visualise it. 

*Dash*<br>
Dash is a Python framework for building web applications. We can write html elements using python and also build Plotly charts.
Plotly is a charting library built on top of d3.js and the Python API allows us to plot graphs in python. See [plotly video][plotly-video]<br><br>
I refer to the [dash user guide][dash-user-guide] to get started. 
A dash app has 2 parts. See [dash video][dash-video]. <br><br> 
**app.layout**<br>
The first one is how the app looks like. We modify it by setting 
app.layout equals to a tree of components like html.Div and dcc.Graph.
The html in html.Div refers to the dash_html_components library which we import.
When we write ``html.Div(children='hello')``, ``<div>hello</div>`` html element is generated.
By default, the first atribute is children. So we could have writen html.Div('hello'). 
There are other components for example html.H1. We can also include the style using a dictionary:<br>
``html.Div(children='hello',style={'textAlign':'center'})``<br><br>
The dcc in dcc.Graph refers to the dash_core_components library.
They contain higher level components that are interactive like dropdowns and radio buttons.
Also the plotly graphs are accessed through this library by ``dcc.Graph``.
The figure variable in dcc.Graph is a dictionary with keys 'data' and 'layout'
and data is a list of trace. A trace in plotly terms means a single variable of data. For example,
if we have 2 line charts in one graph, we have data=[trace1,trace2]. We access the various types of plotly charts
through ``plotly.graph_objs as go`` and call go.Scatter for example.
A list comprehension can be put to good use in generating multiple traces/go.Scatter. For example adapted from the user guide:
{% highlight python %}
data= [
	go.Scatter(
	x=df[df['state'] == i]['gdp per capita'], 
	y=df[df['state'] == i]['life expectancy'],
	name=i
	) for i in df.state.unique()
]
{% endhighlight %}


The layout is a dictionary which we can call by go.Layout() or dict().

We make use of what we have discussed so far in the following app.py script. This script is based on the [plotly video][plotly-video] video link.
We go to the directory where the app.py script is saved in cmd, type python app.py and go to the localhost url to view the chart.

*app.py*
{% highlight python %}
import dash
import dash_core_components as dcc
import dash_html_components as html
import plotly.graph_objs as go

app = dash.Dash(__name__)

x = [1,2,3,4,5,6,7,8,9]
y = [1,2,3,4,0,4,3,2,1]
z = [10,9,8,7,6,5,4,3,2,1]

app.layout = html.Div(children=[
	html.H1(children='hello guys',
	style={'textAlign':'center'}
	),
	html.Div(children='today is friday'),
	dcc.Graph(
	figure = {
		'data': [
		go.Scatter(x=x,y=y,name='list object',line=dict(width=5)),
		go.Scatter(x=x,y=z,name='list object 2',line=dict(width=10))
		],
		'layout':go.Layout(
		title='double chart',xaxis=dict(title='xaxis'),yaxis=dict(title='yaxis')
		)
	}
	)
])

if __name__ == '__main__':
    app.run_server(debug=True)
{% endhighlight %}
<br><br>

<img src="https://raw.githubusercontent.com/noaihere/blog/gh-pages/img/linechart.JPG" class ='img-responsive'><br>
The chart contains 2 lines in 1 chart with a header 'hello guys' and 'today is friday' below it.<br>

**sgparkwhere layout**<br>
The sgparkwhere application has 4 main components in the app layout. 
The first is for the text. These are implemented using html.Div and html.H3.
The second is the component which allows users to type or select a location. This is achieved using dcc.Dropdown.
The third component is the map and the last component is the table. Amyoshino's implementation of NYC Wi-Fi Hotspots 
was a major source of reference for these last 2 components. See [Amyoshino Video][Amyoshino-Video] and [Amyoshino Code][Amyoshino-Code].
<br>The map component is implemented using dcc.Graph. In plotly, there is a graph with type scattermapbox that allows you to display
Mapbox maps. I referred to [plotly mapbox][plotly-mapbox] for an example plotly graph.
The table is implemented using a module called dash_table. See [dash table][dash-table].<br><br>

**@app.callback**<br>
The second part of a dash app is the interactivity using dash callbacks.
Dash callbacks are decorators in python. For more information on decorators, see [decorators][decorators].
For more informatation on how dash implements callbacks see dash getting started part 2, [dash decorators][dash-decorators]
and [def callback][def-callback].
We specify the output with its ID and property, a list of inputs with its ID and property, and a function.
When the input property changes, the function that the callback decorator wraps will get called automatically.
Dash gives the function the new value of the input property as an argument, and updates the property of the output component
with what is returned by the function. 
When dash app starts, it automatically calls all of the callbacks with initial values of input components to output the initial state of the output components.
<br>

{% highlight python %}
import dash
import dash_table
import pandas as pd
import dash_core_components as dcc
import dash_html_components as html
from dash.dependencies import Input, Output

df = pd.read_csv('https://raw.githubusercontent.com/plotly/datasets/master/solar.csv')

app = dash.Dash(__name__)

app.layout =  html.Div([
    dcc.Dropdown(
    id='placecol',
    options=[{'label': i, 'value': i} for i in df['State'].unique()],
    value=''
    ),
	dash_table.DataTable(
    id='table',
    columns=[{"name": i, "id": i} for i in df.columns],
    data=df.to_dict("rows"),
)
])

@app.callback(
    Output(component_id='table', component_property='data'),
    [Input(component_id='placecol', component_property='value')]
)
def update_data(input_value):
	if (input_value!=''):

		df_temp = df.copy()
		df_temp = df_temp[(df_temp.State == input_value)]
		data = df_temp.to_dict("rows")
		
	else:
		data =df.to_dict("rows")
	return data

if __name__ == '__main__':
    app.run_server(debug=True)
{% endhighlight %}
<br>
<img src="https://raw.githubusercontent.com/noaihere/blog/gh-pages/img/filter.JPG" class ='img-responsive'><br>
The code above is adapted from the datatable link above.
When you choose a state in the dropdown, the table is filtered to show only data belonging to that state.<br>



**sgparkwhere interactivity**<br>
For the application, there are 2 callbacks.
The first filters the table shown by the value entered/selected in the dropdown.
The second changes the map shown depending on the data in the table. Both the map's data and layout can be changed.
<br>

The final piece of the puzzle is to be able to constantly get new data from the LTA API and constantly update the map and table.
We do this by using the concept of events. As stated in [def callback][def-callback], we can cause a callback to be called not just by subscribing
to input components. We can subscribe to an event and the callback will be called whenever the event is fired.
I referred to examples [event1][event1] and [event2][event2].
I add to the app.layout a component [dcc.Interval][dcc.Interval] and specify the interval to be 20s.
{% highlight python %}
        dcc.Interval(
            id='interval-component',
            interval=20*1000, # in milliseconds
        )
{% endhighlight %}
Then I modify the 2 callbacks so that they listen to this event. And add to the first callback function,
dataframe = output of api call, so the api will be called and dataframe refreshed every 20s.
{% highlight python %}
@app.callback(
    dash.dependencies.Output('table', 'data'),
    [dash.dependencies.Input('placecol', 'value')],events=[dash.dependencies.Event('interval-component', 'interval')])
def update_selected_loc(value):
	df=getapidata()
	...
{% endhighlight %}

<br>
**Conclusion**<br>
I have covered the main aspects of how the app was built. 
We went from data acquisition to transformation to visualisation. 
Dash, plotly, pandas are useful tools in python to quickly create a data-driven app.
In the next blog, I will cover its deployment to heroku. Thanks for reading.



[app-link]: https://sgparkwhere.herokuapp.com/
[link]: https://www.mytransport.sg/content/dam/datamall/datasets/LTA_DataMall_API_User_Guide.pdf
[namedtuple-link]:https://pymotw.com/2/urlparse/
[namedtuple-link2]:https://pymotw.com/2/collections/namedtuple.html#collections-namedtuple
[geturl-link]:https://docs.python.org/2/library/urllib.html
[json-vid]:https://www.youtube.com/watch?v=9N6a-VLBa2I
[pandas-link]:https://www.shanelynn.ie/select-pandas-dataframe-rows-and-columns-using-iloc-loc-and-ix/
[plotly-video]:https://www.youtube.com/watch?v=B911tZFuaOM
[dash-user-guide]:https://dash.plot.ly/getting-started
[dash-video]:https://www.youtube.com/watch?v=NUXUmv-aeG4
[plotly-mapbox]:https://www.youtube.com/watch?v=NUXUmv-aeG4
[Amyoshino-Video]:https://www.youtube.com/watch?v=lu0PtsMor4E
[Amyoshino-Code]:https://github.com/amyoshino/Dash_Tutorial_Series/blob/master/ex4.py#L87
[dash-table]:https://dash.plot.ly/datatable
[decorators]:https://www.youtube.com/watch?v=FsAPt_9Bf3U
[dash-decorators]:https://medium.com/@plotlygraphs/introducing-dash-5ecf7191b503
[def-callback]:https://github.com/plotly/dash/blob/master/dash/dash.py
[event1]:https://pythonprogramming.net/live-graphs-data-visualization-application-dash-python-tutorial/
[event2]:https://github.com/plotly/dash/issues/147
[dcc.Interval]:https://dash.plot.ly/live-updates