# startup-dashboard
import streamlit as st
import pandas as pd
import matplotlib.pyplot as plt
import plotly.express as px

st.set_page_config(layout='wide', page_title='Startup Analysis')

df = pd.read_csv('startup_final.csv')
df['date'] = pd.to_datetime(df['date'],errors='coerce')
df['year'] = df['date'].dt.year
df['month'] = df['date'].dt.month

def load_overall_analysis():
    st.title('Overall Analysis')

    # total amount invested
    total = round(df['amount'].sum())
    # maximum amount of funding in a startup
    max_fund = df.groupby('startup')['amount'].max().sort_values(ascending= False).head(1).values[0]
    # average amount of funding in a startup
    avg_fund = round(df.groupby('startup')['amount'].sum().mean(), 2)
    # total funded startups
    t_funded = df['startup'].nunique()
    col5,col6,col7,col8 =st.columns(4)
    with col5:
        st.metric('Total Amount', str(total) + ' Cr')

    with col6:
        st.metric('Maximum Funding', str(max_fund) + ' Cr')

    with col7:
        st.metric('Average Funding', str(avg_fund) + ' Cr')

    with col8:
        st.metric('Total Funded Startups', t_funded)

    # Month on month investment
    st.header('MoM Investment Graph')
    selected_type = st.selectbox('Select Type',['Sum','Count'])

    if selected_type == 'Sum':
        temp_df = df.groupby(['year', 'month'])['amount'].sum().reset_index()
        temp_df['x-axis'] = temp_df['month'].astype(str) + '-' + temp_df['year'].astype(str)

        fig6 = px.line(temp_df, x=temp_df['x-axis'], y=temp_df['amount'])
        st.plotly_chart(fig6)
    else:
        temp_df2 = df.groupby(['year', 'month'])['amount'].count().reset_index()
        temp_df2['x-axis'] = temp_df2['month'].astype(str) + '-' + temp_df2['year'].astype(str)

        fig7 = px.line(temp_df2, x=temp_df2['x-axis'], y=temp_df2['amount'])
        st.plotly_chart(fig7)

    # Sector wise investments
    st.header('Top 5 Sector Wise Investments')
    selected_option = st.selectbox('Select Type',['Total','Count'])
    if selected_option == 'Total':
       sector_series1= df.groupby('vertical')['amount'].sum().sort_values(ascending=False).head()

    else:
        sector_series1 = df.groupby('vertical')['amount'].count().sort_values(ascending=False).head()

    fig8 = px.pie(sector_series1, values=sector_series1.values, names=sector_series1.index)
    fig8.update_traces(textposition='inside', textinfo='percent+label')
    st.plotly_chart(fig8)

    # Type of funding
    st.header('Top 5 Types of Funding (Cr)')
    type_series = df.groupby('round')['amount'].count().sort_values(ascending=False).head()
    fig9 = px.bar(type_series, x=type_series.index, y=type_series.values, color=type_series.index)
    st.plotly_chart(fig9)

    # City Wise funding
    st.header('Top 5 Cities Invested In')
    city_sr = df.groupby('city')['amount'].count().sort_values(ascending=False).head()
    fig10 = px.bar(city_sr, x=city_sr.index, y=city_sr.values, color=city_sr.index)
    st.plotly_chart(fig10)

    # Top 5 startups
    st.header('Top 5 Startups')
    top_st = df.groupby('startup')['amount'].sum().sort_values(ascending= False).head()
    fig11 = px.bar(top_st, x=top_st.index, y=top_st.values, color=top_st.index)
    st.plotly_chart(fig11)

    # Top 5 investors
    st.header('Top 5 Investors')
    top_inv = df.groupby('investors')['amount'].sum().sort_values(ascending=False).head()
    fig12 = px.bar(top_inv, x=top_inv.index, y=top_inv.values, color=top_inv.index)
    st.plotly_chart(fig12)




    


def load_investor_details(investor):
    st.title(investor)
    # load the recent five investments made by the investor
    last5_df = df[df['investors'].str.contains(investor)].head()[['date', 'startup', 'vertical', 'city', 'round', 'amount']]
    st.subheader('Most Recent Investments')
    st.dataframe(last5_df)

    col1, col2 = st.columns(2)
    # biggest investments made by the investors
    with col1:
        big_series = df[df['investors'].str.contains(investor)].groupby('startup')['amount'].sum().sort_values(ascending=False).head()
        st.subheader('Biggest Investments')
        #st.dataframe(big_series)
        #fig, ax = plt.subplots()
       # ax.bar(big_series.index,big_series.values)
        #st.pyplot(fig)

        fig1 = px.bar(big_series, x=big_series.index, y=big_series.values, color=big_series.index)
        st.plotly_chart(fig1)

        # generally invests in
    with col2:
        generally_series = df[df['investors'].str.contains(investor)].groupby('vertical')['amount'].sum()
        st.subheader('Sectors Invested In')
        fig2 = px.pie(generally_series, values=generally_series.values, names = generally_series.index)
        fig2.update_traces(textposition='inside', textinfo='percent+label')
        st.plotly_chart(fig2)

    col3, col4 = st.columns(2)
     # rounds invested in
    with col3:
        round_series = df[df['investors'].str.contains(investor)].groupby('round')['amount'].sum()
        st.subheader('Rounds Invested In')
        fig3 = px.pie(round_series, values=round_series.values, names=round_series.index)
        fig3.update_traces(textposition='inside', textinfo='percent+label')
        st.plotly_chart(fig3)
    with col4:
        city_series = df[df['investors'].str.contains(investor)].groupby('city')['amount'].sum()
        st.subheader('Cities Invested In')
        fig4 = px.pie(city_series, values=city_series.values, names=city_series.index)
        fig4.update_traces(textposition='inside', textinfo='percent+label')
        st.plotly_chart(fig4)




    year_series = df[df['investors'].str.contains(investor)].groupby('year')['amount'].sum()
    st.subheader('YoY investment')
    fig5 = px.line(year_series, x=year_series.index, y=year_series.values)
    st.plotly_chart(fig5)


def load_startup_details(startup):
    st.title(selected_startup)
    # load startup details like vertical, subvertical, city
    st_dt = df[df['startup'].str.contains(selected_startup)][['startup', 'vertical', 'subvertical', 'city']].head(1)
    st.subheader("Startup Details")
    st.dataframe(st_dt)

    #load startup rounds pie chart

    st_round = df[df['startup'].str.contains(selected_startup)].groupby('round')['round'].value_counts()
    st.subheader('Startup Rounds Percentage')
    fig13 = px.pie(st_round, values=st_round.values, names=st_round.index)
    fig13.update_traces(textposition='inside', textinfo='percent+label')
    st.plotly_chart(fig13)

    #Top 5 investors of this startup
    startup_inves = df[df['startup'].str.contains(selected_startup)].groupby('investors')['amount'].sum().head().sort_values(ascending=False)
    st.subheader('Top Investors')
    fig14 = px.bar(startup_inves, x=startup_inves.index, y=startup_inves.values, color=startup_inves.index)
    st.plotly_chart(fig14)

    #YoY investment line graph
    YoY_inves = df[df['startup'].str.contains(selected_startup)].groupby('year')['amount'].sum()
    st.subheader('YoY Investment')
    fig15 = px.line(YoY_inves, x=YoY_inves.index, y=YoY_inves.values)
    st.plotly_chart(fig15)





st.sidebar.title('Startup Funding Analysis')

option = st.sidebar.selectbox('Select One', ['Overall Analysis', 'Startup', 'Investor'])

if option == 'Overall Analysis':
   #btn0 = st.sidebar.button('Show Overall Analysis')
   #if btn0:
   load_overall_analysis()


elif option == 'Startup':
    selected_startup = st.sidebar.selectbox('Select Startup', sorted(df['startup'].unique().tolist()))
    btn1 = st.sidebar.button('Find startup details')
    if btn1:
        load_startup_details(selected_startup)
    #st.title('Startup Analysis')

else:
    selected_investor = st.sidebar.selectbox('Select Investor', sorted(set(df['investors'].str.split(',').sum())))
    btn2 = st.sidebar.button('Find investors details')
    if btn2:
        load_investor_details(selected_investor)
    #st.title('Investors Analysis')
