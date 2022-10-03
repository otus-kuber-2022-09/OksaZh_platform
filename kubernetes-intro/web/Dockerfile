FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
RUN chown -R 1001 /etc/nginx/
RUN mkdir -p /run/nginx
USER 1001
EXPOSE 8000
RUN nginx -t
CMD ["nginx", "-g", "daemon off;"]





