from pandas.core.tools.datetimes import to_datetime
import requests
import json
import numpy as np
import pandas as pd
import math
import datetime
import time
from datetime import date
from datetime import timedelta
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from email.mime.application import MIMEApplication


today = date.today()

start_date=today-timedelta(days=30)
today = str(today)
start_date=str(start_date)
from __future__ import division

url = 'https://domain/login'
body = {'username':'','password':'','company':'infinite ventures','device':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.5938.132 Safari/537.36','location':'0,0','outlet':{'node_id':1,'node_type':'Outlet','node_name':'Head Office'}}
headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.5938.132 Safari/537.36','content-type': 'application/json'}
r1 = requests.post(url, data=json.dumps(body), headers=headers)
print(r1)
print(r1.status_code)
data1=r1.json()
jwt = data1['jwt']


url = 'https://domain/sales_reports_bill_wise_Details'
body = {'query':{'search':'','order':{'active':'bid','direction':'desc'},'to':'2023-09-27T12:51:46.855Z','from':'2023-09-26T12:51:46.855Z','toFormated':today ,'formFormated':start_date,'page':1,'limit':100000,'length':2088,'BillNo':'','Customer':'','SalesPerson':'','Suppliers':'','product':'','salesChannel':['POS','INSTAGRAM','WHATSAPP','FACEBOOK','PHONE','VIBER','EMAIL','WEB'],'paymentype':[{'Payment Type':'CASH'},{'Payment Type':'CREDIT'},{'Payment Type':'DEPOSIT'},{'Payment Type':'CARD'},{'Payment Type':'CASH,CARD'},{'Payment Type':'DEPOSIT,CASH'},{'Payment Type':'CASH,DEPOSIT'},{'Payment Type':'CARD,CASH'},{'Payment Type':'CHEQUE'},{'Payment Type':'{\'CREDIT\':\'4990\'}'},{'Payment Type':'{\r\n\t\'DEPOSIT\': \'10950\',\r\n\t\'CREDIT\': \'250\'\r\n}'},{'Payment Type':'EXCHANGE'},{'Payment Type':'ONLINE'},{'Payment Type':''},{'Payment Type':'VOUCHER'},{'Payment Type':'VOUCHER,VOUCHER'},{'Payment Type':'CREDIT,CARD'},{'Payment Type':'VOUCHER,CARD'},{'Payment Type':'DEPOSIT,CARD'},{'Payment Type':'VOUCHER,DEPOSIT'},{'Payment Type':'VOUCHER,VOUCHER,VOUCHER,VOUCHER,VOUCHER,CASH'},{'Payment Type':'VOUCHER,CASH'},{'Payment Type':'CARD,CARD'}],'warehouse':[]}}
headers = {'Authorization':jwt,'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/117.0.5938.63 Safari/537.36','content-type': 'application/json'}
r = requests.post(url, data=json.dumps(body), headers=headers)
print(r)
print(r.status_code)
data=r.json()

df = pd.DataFrame(data['table'])
def calculate_discounted_sales(row):
    if row['Customer'] in ['#C1570 - Uber Eats', '#C1183 - PickMe'
, '#C0035 - Daraz - Protecto'
, '#C0032 - Daraz -  Warehouse Club'
,'#C0367 - Daraz - Polo Store']:
        return row['Sales Amount'] * 0.8
    elif row['Customer'] in ['#C1570 - Uber Eats']:
        return row['Sales Amount'] * 0.86
    else:
        return row['Sales Amount']

# Apply the function to create a new column 'Discounted Sales'
df['After_comission'] = df.apply(calculate_discounted_sales, axis=1)

zero_cost_mask = df['Cost Amount'] == 0

# Calculate Profit Margin using vectorized operations
df.loc[~zero_cost_mask, 'Profit Margin'] = (df['After_comission'] - df['Cost Amount'])*100 / df['Cost Amount']
df.loc[zero_cost_mask, 'Profit Margin'] = 0  # Set Profit Margin to 0 where 'Cost Amount' is zero
profit_margin_less_than_5 = df[df['Profit Margin'] < 5]
# Assuming you want to remove 'Column1' and 'Column2'
columns_to_remove = [ 'Cost Amount','Sales Amount','Location','Address','Created By','City','Exchange Ref','Contact No1','Contact No2','Sales Person','Invoice Type','Reference','Delivery Amount','Cost Price','Sales Price','Discount','Exchange','Sold Qty','Paid By']
profit_margin_less_than_5 = profit_margin_less_than_5.drop(columns=columns_to_remove)

# Assuming 'Date' is the column containing the date
yesterday = date.today() - timedelta(days=1)

print("Yesterday:", yesterday)

yesterday_data = profit_margin_less_than_5[profit_margin_less_than_5['Date'] == yesterday]

# Assuming 'Profit Margin' is the column you want to include in the email body
yesterday_profit_margin = yesterday_data['Profit Margin']

# Convert the series to a string
yesterday_data_str = yesterday_profit_margin.to_string(index=False)

print("Yesterday Data:")

profit_margin_less_than_5['Date'] = pd.to_datetime(profit_margin_less_than_5['Date'])
yesterday = date.today() - timedelta(days=1)
yesterday_data = profit_margin_less_than_5[profit_margin_less_than_5['Date'].dt.date == yesterday]
print(yesterday_data)

print(profit_margin_less_than_5)
profit_margin_less_than_5.to_excel('my_data.xlsx', index=False)


# Define email parameters
sender_email = ''
receiver_emails = ['receiver1 ','receiver']
subject = 'Profit Margin Less Than 5 Report'
# Use it in the email body
yesterday_data_str = yesterday_data.to_string(index=False)

# Include the string in the email body
body = f"Please find the attached report for {yesterday}.\n\n{yesterday_data_str}"


# Create a MIMEText object for the email body
message = MIMEMultipart()
message['From'] = sender_email
message['To'] = ', '.join(receiver_emails)
message['Subject'] = subject
message.attach(MIMEText(body, 'plain'))

# Attach the Excel file
with open('my_data.xlsx', 'rb') as attachment:
    part = MIMEApplication(attachment.read(), Name='my_data.xlsx')
    part['Content-Disposition'] = 'attachment; filename="my_data.xlsx"'
    message.attach(part)

# Send the email
with smtplib.SMTP('smtp.gmail.com', 587) as server:
    server.starttls()
    server.login(sender_email, 'wzlascldkwfxayaz')  # Use an App Password or environment variables for security
    server.sendmail(sender_email, receiver_emails, message.as_string())
