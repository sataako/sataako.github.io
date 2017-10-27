# Sataako?<img src="https://raw.githubusercontent.com/sataako/sataako.github.io/master/images/umbrella-580061.svg?sanitize=true" width="50px" />
The Sataako? -service issues warnings to users of when it will rain at their location within the next hour in the Helsinki metropolitan area. Users can currently access the service via a Telegram bot; other clients like mobile apps can be created by utilising the open API of the fmio-server. The service uses optic flow to interpolate the movement of precipitation systems during the next 60 minutes based on data gathered through FMI open data.

## Example
User is commuting to work with a bike and is about to leave work. He has flexible work hours and does not want to get wet. He will open telegram and start a conversation with @sataakobot on Telegram, give his current location to the bot through pressing a button in the conversation. He will then start to receive telegram messages in the chat when it is about to rain in the next hour at that location and can safely bike home before it is raining. At any time he can exit the chat to stop the notifications.

## Data
Our service uses the Finnish Meteorological Institute (FMI) open data, namely radar data. The radar data used in this project is a rainrate composite of all FMI weather radars and is provided in GeoTIFF format. It is licensed under Creative Commons Attribution 4.0 International. We have limited access to data, so it can't be fetched on every request and must be cached. The access is limited for example by the amount of requests to FMI.

For visualization we also use simple vector map data by Natural Earth in 1:10 000 000 resolution. The data is licensed under Public Domain. We use country border and city location data as a basemap when visualizing rain rate products.

## Data analysis
To be able to give a rain forecast (nowcast), we need to do time extrapolation of radar rain rate fields. This is done using an optic flow algorithm (http://scholarpedia.org/article/Optic_flow). In addition, we also need to do data type conversion and some smoothing of the extrapolated rain rate fields. Given the user coordinates, we analyze if precipitation systems will move above the user location. From the extrapolated precipitation field we calculate the precipitation accumulation in millimeters in the user location. We use this information to issue warnings to the user in case it is about to rain in their location.

The list of recent radar images is queried from the FMI Web Feature Service (WFS). The result of the query is a fairly complicated XML file containing various metadata about the radar images. It is parsed to get links to the data files and corresponding timestamps. We use two most recent radar images to calculate the movement vector field that is used for time extrapolation of the precipitation systems.

## Design
The service uses server-client model. The amount of requests to FMI can be limited, so the polling must be adjustable to fit the limits. The generation of forecasts can be a slow process, so we should do it only when new data is available. The server will actively poll FMI for new radar data. If new data is found, then new forecasts and images are generated based on the data. The clients can poll the server for new forecast data periodically and request images of the forecasts.

## Communication of results
The warnings will be issued as Telegram messages, using Telegram bot API. The user will be informed if there is a risk of rain within the next 60 minutes. The warnings also give information about the estimated precipitation accumulation during the forecast period. User can view animation of weather maps in the Southern Finland and forecasts of rain movement.

<video style="width: 100%" controls=""><source src="https://github.com/sataako/sataako.github.io/blob/master/images/telegram_bot_demo.mp4?raw=true" type="video/mp4"></video>
