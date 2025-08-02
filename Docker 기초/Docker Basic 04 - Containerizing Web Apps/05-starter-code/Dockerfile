## 1. Which base image do you want to use?
FROM python:3.8-slim

## 2. Set the working directory
WORKDIR /app

## 3. Copy the project files into the working directory.
## -> Copy the requirements.txt into the working directory.
COPY flask-demo/requirements.txt . 

## 4. Install the dependencies
RUN pip install -r requirements.txt

## 5. (Added) Copy th remaining project files into the working directory.
COPY flask-demo/ .

## 6. Document and inform the developer that the application will use PORT 5000 of the container.
EXPOSE 5000

## 7. Set an environment shell variable 
## and Define the command to run when the container starts.
ENV FLASK_APP=app.py 
CMD ["flask", "run", "--host=0.0.0.0"]

# (as-is)
## 7. Define the command to run when the container starts.
# CMD ["python", "app.py"]