FROM nginx:alpine
LABEL org.opencontainers.image,title="TEST-WEB"
ENV APP_ENV=dev
COPY app/ /user/share/nginx/html/
EXPOSE 80
HEALTHCHECK --interval=30s --timeout-30s CMD wget -q -0 /dev/null http://localhost // exit 1
