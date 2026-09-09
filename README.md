# demand-forecasting-hvac
Making a Data Science model that grabs data from LADBS and helps forecast the counts of upcoming HVAC permits 6 weeks ahead.

## limitations

- Since 67is-svtd is a live data set, 2020-present, everytime we re run it there is going to be a new total amount of records that shows up. This can be a problem for two reasons one is because of reproducibility. If you run the data today and then run it tomorrow you will get different predictions. You can get partial weeks because of this live api, lets say you a are training on older data then you run your model when you check it against the partial week its will seem like the value you got was way off but in actuallty a since it was a partial week thursday and friday hadent been filed yet. So you would think your model is way worse than it actually is.
