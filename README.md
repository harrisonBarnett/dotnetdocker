Welcome to the quick and simple idiot's guide to hosting a containerized ASP.NET Core application on Docker. I just used the default ASP.NET WeatherForecast API. The repository for this can be found [here](https://github.com/harrisonBarnett/dotnetdocker)

Make your app. Do whatever you want with it, but understand that if you want to prop up a database that we'll have to go over that in another guide.

At any rate, if you're building your app from scratch, you can choose to add Docker support from the dialogs while you're giving your app a name, disabling HTTPS, etc. Make sure the `Dockerfile` is located in the same directory as your `.csproj` file. 

You will likely want to choose linux as the target for the Docker image that will be running your app, because who the fuck wants to run an entire Windows isntance? This matters because of PATH stuff. Just do it.

If your app has already been built, you can right click the project, select **Add>Docker Support...**

Both of these techniques will generate a generic Dockerfile, but we want to make some changes. Make your Dockerfile look like this:

**Dockerfile**
```
# select the build environment for your application
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build-env
# make a working directory in your container where all your shit will end up
WORKDIR /app

# Copy everything to the container
COPY . ./
# Restore .NET project dependencies
RUN dotnet restore
# Build and publish a release (good for iteration)
RUN dotnet publish -c Release -o out

# Build runtime image (in this case the ASP.NET framework)
FROM mcr.microsoft.com/dotnet/aspnet:6.0
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "<your_app_name>.dll"]
```
*NB: the .dll should live in your project's bin/Release/net6.0/ directory*

### before running any docker commands
Build a release version of your application. Doesn't matter if it's a prototype or whatever, just do it. It will rebuild a new release every time we make a new docker image anyway, but we'll need a release **.dll** initially.

In the directory containing your `.csproj` file, run this in a terminal: `dotnet publish -c Release `. Boom, you now have an executable of your app.

### docker build commands
Build your image:
`docker build -t <your_img_name> -f Dockerfile . --no-cache`
where:
- t = the name of your image
- f = the Dockerfile you'll be building from
- --no-cache = ensures a clean build every time

Confirm your image exists:
`docker images`

Instantiate your container:
`docker create -p 8081:80 --name <some_container_name> <your_img_name>`
*NB: the order of operations here matters!*\
where:
- p = the port you'd like to map. the container will have its own port defaulted to :80, which you can map to whatever you want (in this case 8081) so that you can access the container from "the outside"
- --name = lets you give your container a descriptive name


### helpful links:
msdn on containerizing .net apps:
https://docs.microsoft.com/en-us/visualstudio/containers/container-build?view=vs-2019&WT.mc_id=visualstudio_containers_aka_containerfastmode
