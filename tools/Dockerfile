FROM debian

WORKDIR /github

RUN set -eux; \
        apt update; \
        apt install -y ruby-full build-essential zlib1g-dev git; \
        echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc; \
        echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc; \
        gem install jekyll bundler

ENTRYPOINT bundle install && tools/run.sh
