# Sataako?<img src="https://raw.githubusercontent.com/sataako/sataako.github.io/master/images/umbrella-580061.svg?sanitize=true" width="50px" />
The Finnish weather is known for its variability, and is rightfully one of the most common topics of smalltalk. Often in everyday life, you find yourself wondering “will it rain?” (“Sataako?” in Finnish). Should you take the bike or the tram. Many professionals, like farmers, also strongly rely on this information.

The Sataako? -service issues warnings to users of when it will rain at their location within the next hour in Southern Finland. Users can currently access the service via a Telegram bot; other clients like mobile apps can be created by utilising the open API of the data server of the service, titled fmio-server. The service uses a motion algorithm to forecast the movement of precipitation systems during the next 60 minutes based on weather radar data gathered through FMI open data.

## Use case
Terhi commutes by bike and is about to head home. The weather looks like it might rain soon and she does not want to get wet. She opens Telegram and starts a conversation with `@sataakobot`. She gives her current location to the bot through pressing a button in the conversation. By doing this she has subscribed to Telegram alerts from the bot in case it’s about to rain within the next hour at her location. The bot gives a warning saying it will start raining in 45 minutes. Given this information she decides she has just enough time to pedal home before the rain. When she has reached home, dry, she disables the notifications since it’s Friday and she does not need the bike during the weekend.

Below you can see a demo video of the app.
<video style="width: 100%" poster="https://github.com/sataako/sataako.github.io/blob/master/images/photo6023965351362013859.jpg?raw=true" controls=""><source src="https://github.com/sataako/sataako.github.io/blob/master/images/telegram_bot_demo.mp4?raw=true" type="video/mp4"></video>

## Data
Our service uses the [Finnish Meteorological Institute (FMI) open data](https://en.ilmatieteenlaitos.fi/open-data), namely radar data. The radar data used in this project is a rain rate composite of all FMI weather radars and is provided in GeoTIFF format. It is licensed under [Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/). We have limited access to data, so it can't be fetched on every request and must be cached. The access is limited for example by the amount of requests to FMI.
 
For visualization we also use simple vector map data by Natural Earth in 1:10 000 000 resolution. The data is published in [public domain](https://en.wikipedia.org/wiki/Public_domain_equivalent_license). We use country border and city location data as a base map when visualizing rain rate products. The GeoTIFF format holds location information for the rain rate data which is used for plotting it on the base map.
 
The data is cropped to Southern Finland after downloading to reduce processing time and resource costs during development.

## Data analysis
To be able to give a rain forecast (nowcast), we need to do time extrapolation of radar rain rate fields. This is done using an [optic flow algorithm](http://scholarpedia.org/article/Optic_flow). In addition, we also need to do data type conversion and some smoothing of the extrapolated rain rate fields. Given the user coordinates, we analyze if precipitation systems will move above the user location. From the extrapolated precipitation field we calculate the precipitation accumulation in millimeters in the user location. We use this information to issue warnings to the user in case it is about to rain in their location.
 
The list of recent radar images is queried from the FMI Web Feature Service (WFS). The result of the query is a fairly complicated XML file containing various metadata about the radar images. It is parsed to get links to the data files and corresponding timestamps. We use two most recent radar images to calculate the movement vector field that is used for time extrapolation of the precipitation systems.

## Design
The service uses the server-client model to separate data collection and analysis from the user clients. The amount of requests to the FMI API is limited but enough to always download the most fresh radar data. The generation of forecasts can be a slow process, so it is done only when new data is available. The server actively polls FMI for new radar data every couple of minutes. If new data is found, new forecasts, images and videos of the forecasts are generated based on the data. FMI normally generates the radar rain rate composite product every 5 minutes, but our data server is designed to handle short data gaps. The clients can poll the server for new forecast data periodically and request images of the forecasts.

<img src="https://github.com/sataako/sataako.github.io/blob/master/images/photo6023965351362013861.jpg?raw=true" />

## Communication of results
The warnings will be issued as Telegram messages, using Telegram bot API. The user will be informed if there is a risk of rain within the next 60 minutes. The warnings also give information about the estimated precipitation accumulation during the forecast period. The user can also view an animation of the rain forecast for Southern Finland.
 
## Deliverables
The source code of the service and the bot can be found in [our Github repositories](https://github.com/sataako).
The Telegram bot can be found on [Telegram messenger](https://telegram.org/) by searching for `@sataakobot` or by going directly to the bot in [web view](https://web.telegram.org/#/im?p=@sataakobot). Registration is needed for using Telegram. Both Telegram and the bot are free to use. The bot is not currently running on a dedicated server - don’t be surprised if it appears to be down.
