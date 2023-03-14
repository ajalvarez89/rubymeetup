FROM ruby:3.1
RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
COPY . /usr/src/app/
WORKDIR /usr/src/app
RUN bundle
CMD ["bin/rails", "s", "-b", "0.0.0.0"]
