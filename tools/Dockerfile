FROM ruby:2.7
WORKDIR /github

RUN set -eux; \
        apt update; \
        apt install -y build-essential procps; \
        echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc; \
        echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc; \
        . ~/.bashrc; \
        gem update --system; \
        gem install jekyll bundler

ENTRYPOINT bundle install && tools/run.sh
