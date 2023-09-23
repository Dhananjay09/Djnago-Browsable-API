Browsable API in Django REST Framework
Read
Discuss
Courses
Practice
The browsable API feature in the Django REST framework generates HTML output for different resources. It facilitates interaction with RESTful web service through any web browser. To enable this feature, we should specify text/html for the Content-Type key in the request header. It helps us to use web browsers to surf through the API and can make different HTTP requests. In this section, we will work with the browsable API feature in the Django REST API framework.  

To check how to setup Django RESt Framework and create a API visit – How to Create a basic API using Django Rest Framework ?

Create a Simple Project to Demonstrate Browsable APIs – 
Let’s create models, serializers, and views required for our application robots.

Creating Models
In Django, Models are classes that deal with databases in an object-oriented way. Each model class refers to a database table and each attribute in the model class refers to a database column. Here, we will create the following models:


RobotCategory (Robot Categories)
Manufacturer  (Manufacturer Details)
Robot (Robot Details)
The RobotCategory model requires:

Robot category name
The Manufacturer model requires:

Manufacturer name
The Robot model requires:

Robot name
A foreign key to the RobotCategory model
A foreign key to the Manufacturer model
Currency
Price
Manufacturing date
Let’s look into the HTTP verb, scope semantics in our robots Restful web service.

HTTP Verb	Scope	Semantics	URL
GET	Robot Category	Retrieve a Robot Category	http://localhost:8000/robocategory/{id}/
GET	Collection of Robot Category	Retrieve all Robot Category in the collection	http://localhost:8000/robocategory/
POST	Collection of Robot Category	Create a new Robot Category in the collection	http://localhost:8000/robocategory/{id}/
PUT	Robot Category	Update a Robot Category	http://localhost:8000/robocategory/{id}/
DELETE	Robot Category	Delete a Robot Category	http://localhost:8000/robocategory/{id}/
GET	Manufacturer	Retrieve a Manufacturer 	http://localhost:8000/manufacturer/{id}/
GET	Collection of Manufacturer	Retrieve all Manufacturer in the collection	http://localhost:8000/manufacturer/
POST	Collection of Manufacturer	Create a Manufacturer in the collection	http://localhost:8000/manufacturer/{id}/
PUT	Manufacturer	Update a Manufacturer	http://localhost:8000/manufacturer/{id}/
DELETE	Manufacturer	Delete a Manufacturer	http://localhost:8000/manufacturer/{id}/
GET	Robot	Retrieve a Robot	http://localhost:8000/robot/{id}/
GET	Collection of Robot	Retrieve all Robot in the collection	http://localhost:8000/robot/
POST	Collection of Robot	Create a Robot in the collection	http://localhost:8000/robot/{id}/
PUT	Robot	Update a Robot	http://localhost:8000/robot/{id}/
DELETE	Robot	Delete a Robot	http://localhost:8000/robot/{id}/
Let’s create the models for the robot category, manufacturer, robot, and their relationships. You can add the below code in the models.py file.

from django.db import models
  
class RobotCategory(models.Model):
    name = models.CharField(max_length=150, unique=True)
  
    class Meta:
        ordering = ('name',)
  
    def __str__(self):
        return self.name
  
class Manufacturer(models.Model):
    name = models.CharField(max_length=150, unique=True)
  
    class Meta:
        ordering = ('name',)
  
    def __str__(self):
        return self.name
  
class Robot(models.Model):
    CURRENCY_CHOICES = (
        ('INR', 'Indian Rupee'),
        ('USD', 'US Dollar'),
        ('EUR', 'Euro'),
    )
  
    name = models.CharField(max_length=150, unique=True)
    robot_category = models.ForeignKey(
        RobotCategory,
        related_name='robots',
        on_delete=models.CASCADE)
    manufacturer = models.ForeignKey(
        Manufacturer,
        related_name='robots',
        on_delete=models.CASCADE)
    currency = models.CharField(
        max_length=3,
        choices= CURRENCY_CHOICES,
        default='INR')
    price = models.IntegerField()
    manufacturing_date = models.DateTimeField()
  
    class Meta:
        ordering = ('name',)
  
    def __str__(self):
        return self.name
Here we have three classes that are subclasses of the django.db.models.Model class:

RobotCategory
Manufacturer
Robot
The Robot class holds a many-to-one relationship to the RobotCategory model and Manufacturer model. This relationship is achieved by making use of the django.db.models.ForeignKey class. The code as follows:

    robot_category = models.ForeignKey(
        RobotCategory,
        related_name='robots',
        on_delete=models.CASCADE)
    manufacturer = models.ForeignKey(
        Manufacturer,
        related_name='robots',
        on_delete=models.CASCADE)
The related_name argument creates a reverse relationship. Here the value ‘robots’ in related_name creates a reverse relationship from RobotCategory to Robot and Manufacturer to Robot. This facilitates fetching all the robots that belong to a robot category and also based on the manufacturer.

Next, you can perform the migration process and apply all generated migration. You can use the below commands


python manage.py makemigrations

python manage.py migrate

Creating Serializers
Now, we need to serialize the RobotCategory, Manufacturer, and Robot instances. Here, we will use HyperlinkedModelSerializer to deal with the model relationships. You can check DRF Serializer Relations topic to understand in detail.

from rest_framework import serializers
from robots.models import RobotCategory, Manufacturer, Robot
  
class RobotCategorySerializer(serializers.HyperlinkedModelSerializer):
    robots = serializers.HyperlinkedRelatedField(
        many=True,
        read_only=True,
        view_name='robot-detail')
  
    class Meta:
        model = RobotCategory
        fields = '__all__'
  
  
class ManufacturerSerializer(serializers.HyperlinkedModelSerializer):
    robots = serializers.HyperlinkedRelatedField(
        many=True,
        read_only=True,
        view_name='robot-detail')
  
    class Meta:
        model = Manufacturer
        fields = '__all__'
  
class RobotSerializer(serializers.HyperlinkedModelSerializer):
    robot_category = serializers.SlugRelatedField(
        queryset=RobotCategory.objects.all(), slug_field='name')
    manufacturer = serializers.SlugRelatedField(
        queryset=Manufacturer.objects.all(), slug_field='name')
    currency = serializers.ChoiceField(
        choices=Robot.CURRENCY_CHOICES)
    currency_name = serializers.CharField(
        source='get_currency_display',
        read_only=True)
  
    class Meta:
        model = Robot
        fields = '__all__'
The RobotCategorySerializer and ManufacturerSerializer class are subclasses of HyperlinkedModelSerializer class, and the reverse relationship (RobotCategory to Robot and Manufacturer to Robot) is represented using HyperlinkedRelatedField with many and read_only attributes set to True. The view_name — robot-detail — allows the browsable API feature to provide a click facility for the user, to render the hyperlink. 

The RobotSerializer class is also a subclass of HyperlinkedModelSerializer class. The RobotSerializer class declares two attributes – robot_category and manufacturer – that holds an instance of serializers.SlugRelatedField. A SlugRelated Field represents a relationship by a unique slug attribute.

Creating Views
Let’s make use of generic class-based views provided by Django REST Framework to process the HTTP requests and to provide the appropriate HTTP responses. You can check DRF Class based views for a detailed explanation.

from django.shortcuts import render
  
from rest_framework import generics
from rest_framework.response import Response
from rest_framework.reverse import reverse
  
from robots.models import RobotCategory, Manufacturer, Robot
from robots.serializers import RobotCategorySerializer, \
     ManufacturerSerializer, RobotSerializer
  
  
class ApiRoot(generics.GenericAPIView):
    name = 'api-root'
    def get(self, request, *args, **kwargs):
        return Response({
            'robot-categories': reverse(RobotCategoryList.name, request=request),
            'manufacturers': reverse(ManufacturerList.name, request=request),
            'robots': reverse(RobotList.name, request=request)
            })    
  
  
class RobotCategoryList(generics.ListCreateAPIView):
    queryset = RobotCategory.objects.all()
    serializer_class = RobotCategorySerializer
    name = 'robotcategory-list'
  
class RobotCategoryDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = RobotCategory.objects.all()
    serializer_class = RobotCategorySerializer
    name = 'robotcategory-detail'
  
class ManufacturerList(generics.ListCreateAPIView):
    queryset = Manufacturer.objects.all()
    serializer_class = ManufacturerSerializer
    name= 'manufacturer-list'
  
class ManufacturerDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Manufacturer.objects.all()
    serializer_class = ManufacturerSerializer
    name = 'manufacturer-detail'
  
class RobotList(generics.ListCreateAPIView):
    queryset = Robot.objects.all()
    serializer_class = RobotSerializer
    name = 'robot-list'
  
class RobotDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Robot.objects.all()
    serializer_class = RobotSerializer
    name = 'robot-detail'
Here, our view classes import from rest_framework.generics and we took advantage of two generic class-based views — ListCreateAPIView and RetrieveUpdateDestroyAPIView.

You can notice an ApiRoot class, a subclass of the generics.GenericAPIView, creates an endpoint for the root of our web service. It facilitates browsing the resources with the browsable API feature. 

class ApiRoot(generics.GenericAPIView):
    name = 'api-root'
    def get(self, request, *args, **kwargs):
        return Response({
            'robot-categories': reverse(RobotCategoryList.name, request=request),
            'manufacturers': reverse(ManufacturerList.name, request=request),
            'robots': reverse(RobotList.name, request=request)
            })    
The get method returns a Response object (as a key/value pair of strings) that has the descriptive name for the view and its URL.

Setting URL Conf
Go to the apps (robots) folder and create a new file named urls.py file. You can add the below code:

from django.urls import path
from robots import views
  
urlpatterns = [
    path('robocategory/',
         views.RobotCategoryList.as_view(),
         name='robotcategory-list'),
    path('robocategory/<int:pk>/',
         views.RobotCategoryDetail.as_view(),
         name='robotcategory-detail'),
    path('manufacturer/',
         views.ManufacturerList.as_view(),
         name='manufacturer-list'),
    path('manufacturer/<int:pk>/',
         views.ManufacturerDetail.as_view(),
         name='manufacturer-detail'),
    path('robot/',
         views.RobotList.as_view(),
         name='robot-list'),
    path('robot/<int:pk>/',
         views.RobotDetail.as_view(),
         name='robot-detail'),
    path('',
        views.ApiRoot.as_view(),
        name=views.ApiRoot.name)
]
It defines the URL pattern that has to be matched in the request to execute the particular method for a class-based view defined in the views.py file. Now we have to set the root URL configuration. You can add the below code:

from django.contrib import admin
from django.urls import path, include
  
urlpatterns = [
    path('', include('robots.urls')),
]
How to Make Requests to API using Browsable API?
Let’s compose and send HTTP requests to generate text/html content in the response. The RESTFul web service uses the BrowsableAPIRenderer class to generate HTML content. The HTTPie command to accept text/html as follows:

http -v :8000/robot/ “Accept:text/html”

Sharing the command prompt screenshot for your reference



Before making use of browsable API, let’s create a new entry for Robot Category, Manufacturer, and Robot using the HTTPie command. The command as follows:

http POST :8000/robocategory/ name=”Articulated Robots”

http POST :8000/manufacturer/ name=”Fanuc”

http POST :8000/robot/ name=”FANUC M-710ic/50″ robot_category=”Articulated Robots” manufacturer=”Fanuc” currency=”USD” price=37000 manufacturing_date=”2019-10-12 00:00:00+00:00″

GET HTTP Request
Now, let’s browse the ‘robots’ Restful web service using a browser. You can use the below URL.

http://localhost:8000/

Sharing the screenshot of the browser for your reference



You can click the link that corresponds to robot-categories, manufacturers, and robots and check the data. Sharing the browser screenshot that displays the result of robots (http://localhost:8000/robot/)



POST HTTP Request
Next, let’s create a new robot category. You can browse the below link and scroll down.

http://localhost:8000/robocategory/

Sharing the browser screenshot for your reference



You can type the name of the new robot category and click the POST button. Here, it displays in HTML form. If you choose Raw data, select media type as application/json, populate the new robot category name to the name field, and click the POST button. Sharing the screenshot for your reference.



Sharing the output screenshot



Let’s create a new manufacturer, you can browse the below URL

http://localhost:8000/manufacturer/

Sharing the browser screenshot



You can type the manufacturer name (ABB) and click the POST button. The browser displays the output as shown below



Finally, let’s create a new entry for the robot. You can browse the below URL and scroll down.

http://localhost:8000/robot/

Let’s populate the data. Sharing the browser screenshot for your reference



Here, you can notice that the Robot category, Manufacture, and Currency are dropdown field. After populating the entries, you can click the POST button. Sharing below the screenshot of the displayed output.



PUT HTTP Request
Let’s edit the price of the robot that has a pk value 2. You can browse the below URL and scroll down. 

http://localhost:8000/robot/2/

Sharing the browser screenshot. You can change the price to 27000 and click the PUT button.



Sharing the screenshot of the output.



DELETE HTTP Request
You can create a new test entry and browse the URL with the pk value. 

http://localhost:8000/robot/2/

You can notice a DELETE button. Sharing the browser screenshot below:



On clicking the Delete button, the browser confirms the same. You can click the Delete button in the confirmation window. Sharing the screenshot below.



If the deletion is successful, it displays the below output.



Let’s Wrap UP
From this section, we understood how to make use of the browsable API feature in the Django REST API framework. We composed and sent HTTP requests that generate text/html content as a response and also analyzed the response in a web browser.
