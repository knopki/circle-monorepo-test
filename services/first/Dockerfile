#
# build image
#
FROM node:10 as build

WORKDIR /opt/

# needed programs
RUN npm install -g yarn diarrhea

# copy configs
ADD package.json yarn.lock ./

# create production (or dev - for local build) node_modules in custom directory
ARG NODE_ENV=production
ENV NODE_ENV=$NODE_ENV
RUN yarn install --no-progress --silent --non-interactive --link-duplicates \
  && diarrhea -n default:safe -P -r \
  && mv node_modules /node_modules_prod

# install dev dependencies - we need build tools
RUN yarn install --no-progress --silent --non-interactive --production=false

# copy source and build AFTER dependencies =>
# we don't need to rebuild deps on every source change
ADD src/ ./src
RUN yarn build

#
# production image
#
FROM node:10-alpine

WORKDIR /opt/

# copy production node_modules from build
COPY --from=build /node_modules_prod/ /opt/node_modules/

# copy yarn config and release sources
COPY --from=build /opt/package.json /opt/yarn.lock /opt/
COPY --from=build /opt/dist/ /opt/dist/

ENTRYPOINT [ "yarn", "start" ]
