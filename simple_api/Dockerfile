FROM python:2.7-buster
LABEL org.opencontainers.image.authors="Abdel-had HANAMI hanami.abdel.had@gmail.com"
RUN apt-get update -y && apt-get install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
RUN pip install flask==1.1.2 flask_httpauth==4.1.0 flask_simpleldap python-dotenv==0.14.0 
COPY ./student_age.py /
EXPOSE 5000
ENTRYPOINT [ "python" ]
CMD [ "./student_age.py" ]

