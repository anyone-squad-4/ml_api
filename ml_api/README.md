# ML API

This code contains the app described at the end of the "Serving ML models with APIs" lecture. It features a model to predict the probability of someone having diabetes and stores the predictions in an SQLite database following a monolithic approach. It also contains a Dockerfile that is used in the examples of the  class "Containerization with Docker".

To run it locally first install the requirements: `pip install -r requirements.txt` and run the app with `fastapi dev main.py`.

To run it using the docker container first build the docker image from the Dockerfile: `docker build -t ml_api .` and the run it binding the desired port and volume: `docker run -p 8000:8000 -v ./data:/app/data --name ml_api ml_api`.

## API endpoints:

- **[GET] /**: Hello world of the api
- **[POST] /predict**: Expects a json object containing a sample as described in (ml_api/routes/model_routes.py)[ml_api/routes/model_routes.py] and returns the prediction of the model
- **[GET] /predictions**: Returns the list of predictions performed by the model.

Also, **/docs** contains the automatic documentation of the API generated by FastAPI.
