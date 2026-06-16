FROM python:3.9 AS builder
WORKDIR /opt/server
COPY payment.ini *.py requirements.txt .
RUN pip3 install --prefix=/install -r requirements.txt


FROM python:3.9.25-alpine3.21
EXPOSE 8080
WORKDIR /opt/server
RUN addgroup -S roboshop && adduser -S roboshop -G roboshop
ENV CART_HOST="cart" \
    CART_PORT="8080" \
    USER_HOST="user" \
    USER_PORT="8080" \
    AMQP_HOST="rabbitmq" \
    AMQP_USER="roboshop" \
    AMQP_PASS="roboshop123"
COPY --from=builder /install /usr/local
COPY --chown=roboshop:roboshop payment.ini *.py requirements.txt .
USER roboshop
CMD ["uwsgi", "--ini", "payment.ini"]



# # FROM python:3.9
# EXPOSE 8080
# WORKDIR /opt/server
# COPY payment.ini .
# COPY requirements.txt .
# COPY *.py .
# RUN pip install -r requirements.txt
# ENV CART_HOST="cart" \
#     CART_PORT="8080" \
#     USER_HOST="user" \
#     USER_PORT="8080" \
#     AMQP_HOST="rabbitmq" \
#     AMQP_USER="roboshop" \
#     AMQP_PASS="roboshop123"
# CMD ["uwsgi", "--ini", "payment.ini"]