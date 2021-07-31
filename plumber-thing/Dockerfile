FROM rocker/tidyverse:latest

LABEL org.opencontainers.image.authors="Andrew Heiss <andrew@andrewheiss.com>"

# Install R packages
RUN install2.r --error --deps TRUE \
    plumber \
    && rm -rf /tmp/downloaded_packages/ /tmp/*.rds

# This ADD line purposely breaks the Docker cache after this point. That way 
# everything gets recopied into the container every time
# https://stackoverflow.com/a/55621942/591574
ADD https://google.com cache_bust

# Move the plumber app folder into the container
COPY /app /app
RUN chmod -R 755 /app

# Run this API on port 8000
EXPOSE 8000
ENTRYPOINT ["R", "-e", "pr <- plumber::plumb('/app/plumber.R'); pr$run(host='0.0.0.0', port=8000)"]
