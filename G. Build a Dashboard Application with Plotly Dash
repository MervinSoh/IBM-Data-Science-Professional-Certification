# Import required libraries
import pandas as pd

if not __name__ == '__main__':
    import jupyter_dash

import dash
import dash_html_components as html
#from dash import html # Alternate to line above in case of newer dash versions
import dash_core_components as dcc
#from dash import dcc # Alternate to line above in case of newer dash versions
from dash.dependencies import Input, Output
import plotly.express as px

# Read the airline data into pandas dataframe
spacex_df = pd.read_csv("spacex_launch_dash.csv")
max_payload = spacex_df['Payload Mass (kg)'].max()
min_payload = spacex_df['Payload Mass (kg)'].min()
landing_sites = spacex_df['Launch Site'].sort_values().unique().tolist()

# Create a dash application
if __name__ == '__main__':
    app = dash.Dash(__name__)
else:
    app = jupyter_dash.JupyterDash(__name__)

# Create an app layout
app.layout = html.Div(children=[html.H1('SpaceX Launch Records Dashboard',
                                        style={'textAlign': 'center', 'color': '#503D36',
                                               'font-size': 40}),
                                # TASK 1: Add a dropdown list to enable Launch Site selection
                                # The default select value is for ALL sites
                                dcc.Dropdown(id='site-dropdown',
                                             options=[{'label':ll, 'value':ll} for ll in [*landing_sites, 'All']],
                                             value='All',
                                             placeholder='Select a Launch Site here',
                                             searchable=True
                                             ),
                                html.Br(),

                                # TASK 2: Add a pie chart to show the total successful launches count for all sites
                                # If a specific launch site was selected, show the Success vs. Failed counts for the site
                                html.Div(dcc.Graph(id='success-pie-chart')),
                                html.Br(),

                                html.P("Payload range (kg):"),
                                # TASK 3: Add a slider to select payload range
                                html.Div(dcc.RangeSlider(id='payload-slider',
                                                          min=0,
                                                          max=10000,
                                                          step=1000,
                                                          value=[min_payload, max_payload],
                                                          marks={x:str(x) for x in range(0,11000,1000)} # str(x) replaces x for compatibility with jupyterlab-dash
                                                          ),
                                ),
                                html.Br(),

                                # TASK 4: Add a scatter chart to show the correlation between payload and launch success
                                html.Div(dcc.Graph(id='success-payload-scatter-chart')),
                                ])

# TASK 2:
# Add a callback function for `site-dropdown` as input, `success-pie-chart` as output
@app.callback(
    Output(component_id='success-pie-chart', component_property='figure'),
    Input(component_id='site-dropdown', component_property='value')
)
def render_pie(input_value):
    if input_value=='All':
        return px.pie(spacex_df, values='class', names='Launch Site', title='Total Successful Launches from All Sites')
    else:
        return px.pie(spacex_df.loc[spacex_df['Launch Site']==input_value,['class','Launch Site']].groupby('class').count().reset_index().rename(columns={'Launch Site':'Count'}), values='Count', names='class', title='Launch Outcomes for Site '+input_value)

# TASK 4:
# Add a callback function for `site-dropdown` and `payload-slider` as inputs, `success-payload-scatter-chart` as output
@app.callback(
    Output(component_id='success-payload-scatter-chart', component_property='figure'),
    [Input(component_id='site-dropdown', component_property='value'),
     Input(component_id='payload-slider', component_property='value')     
    ]
)
def render_scatter(input_site, input_range):
    if input_site=='All':
        df = spacex_df.loc[spacex_df['Payload Mass (kg)'].between(*input_range),:].copy()
        success_rate = df['class'].mean()
        success_count = df['class'].sum()
        total_count = df['class'].count()
        df_booster = df[['Booster Version Category','class']].groupby('Booster Version Category').mean().reset_index().rename(columns={'class':'Success Rate'})
        df_booster['Count'] = df[['Booster Version Category','class']].groupby('Booster Version Category').sum()['class'].values
        df_booster['Total'] = df[['Booster Version Category','class']].groupby('Booster Version Category').count()['class'].values
        df_booster['New Name'] = [f"{row['Booster Version Category']:4s}, {row['Success Rate']*100:5.1f}% ({row['Count']}/{row['Total']})" for i, row in df_booster.iterrows()]
        df.loc[:,'Booster Version Category'] = df.loc[:,'Booster Version Category'].map(dict(zip(df_booster['Booster Version Category'],df_booster['New Name']))).values # added .values and .loc[:] to prevent assigning slice error
        return px.scatter(df,
                          x='Payload Mass (kg)',
                          y='class',
                          color='Booster Version Category',
                          title='All Launch Outcomes by Booster Version for Payload Mass '+str(input_range[0])+'-'+str(input_range[1])+' kg: Success rate '+f"{success_rate*100:.1f}% ({success_count}/{total_count})"
                         )
    else:
        df = spacex_df.loc[(spacex_df['Launch Site']==input_site)
                            & (spacex_df['Payload Mass (kg)'].between(*input_range)),:].copy()
        success_rate = df['class'].mean()
        success_count = df['class'].sum()
        total_count = df['class'].count()
        df_booster = df[['Booster Version Category','class']].groupby('Booster Version Category').mean().reset_index().rename(columns={'class':'Success Rate'})
        df_booster['Count'] = df[['Booster Version Category','class']].groupby('Booster Version Category').sum()['class'].values
        df_booster['Total'] = df[['Booster Version Category','class']].groupby('Booster Version Category').count()['class'].values
        df_booster['New Name'] = [f"{row['Booster Version Category']:4s}, {row['Success Rate']*100:5.1f}% ({row['Count']}/{row['Total']})" for i, row in df_booster.iterrows()]
        df.loc[:,'Booster Version Category'] = df.loc[:,'Booster Version Category'].map(dict(zip(df_booster['Booster Version Category'],df_booster['New Name']))).values # added .values and .loc[:] to prevent assigning slice error
        return px.scatter(df,
                          x='Payload Mass (kg)',
                          y='class',
                          color='Booster Version Category',
                          title='Launch Outcomes by Booster Version for Payload Mass '+str(input_range[0])+'-'+str(input_range[1])+' kg at Launch Site '+input_site+': Success rate '+f"{success_rate*100:.1f}% ({success_count}/{total_count})"
                         )

# Run the app
if __name__ == '__main__':
    app.run_server()
