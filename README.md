reate a simple Django program that integrates Google Maps into a web app.
This simple Django application integrates Google Maps into a web app. Here's a breakdown of the code:


1.In settings.py, we add 'maps' to the INSTALLED_APPS list.
2.In models.py, we define a Location model to store place names and their coordinates.
3.In views.py, we create a view function that fetches all locations and passes them to the template.
4.In urls.py, we set up a URL pattern for the map view.

The map.html template contains the HTML structure and JavaScript code to initialize the Google Map and add markers for each location.

To use this code:

Create a new Django project and app named 'maps'.
Copy the provided code into the respective files.
Run migrations to create the database table for the Location model.
Replace 'YOUR_API_KEY' in the template with a valid Google Maps API key.
Add some Location objects through the Django admin or shell.
Run the Django development server and navigate to the map URL.

This example provides a foundation for integrating Google Maps into a Django application.
Students can expand on this by adding features like user input for new locations, custom styling for the map,
or additional information in marker popups.


# maps/models.py
from django.db import models

class Location(models.Model):
    name = models.CharField(max_length=100)
    latitude = models.FloatField()
    longitude = models.FloatField()

    def __str__(self):
        return self.name

# maps/views.py
from django.shortcuts import render
from .models import Location

def map_view(request):
    # Coordinates for Government Polytechnic College, Cherthala
    cherthala_poly = Location.objects.get_or_create(
        name="Government Polytechnic College, Cherthala",
        latitude=9.6838,
        longitude=76.3354
    )[0]
    
    locations = Location.objects.all()
    return render(request, 'maps/map.html', {'locations': locations, 'center': cherthala_poly})

# maps/templates/maps/map.html
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <title>Google Maps Example</title>
    <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=places"></script>
    <style>
        #map { height: 400px; width: 100%; }
        #search-container { margin-bottom: 10px; }
        #distance { margin-top: 10px; }
    </style>
</head>
<body>
    <h1>Google Maps Integration</h1>
    <div id="search-container">
        <input id="search-input" type="text" placeholder="Search for a location">
        <button onclick="searchLocation()">Search</button>
    </div>
    <div id="map"></div>
    <div id="distance"></div>
    <script>
        var map;
        var markers = [];
        var directionsService;
        var directionsRenderer;

        function initMap() {
            directionsService = new google.maps.DirectionsService();
            directionsRenderer = new google.maps.DirectionsRenderer();

            map = new google.maps.Map(document.getElementById('map'), {
                center: {lat: {{ center.latitude }}, lng: {{ center.longitude }}},
                zoom: 12
            });

            directionsRenderer.setMap(map);

            var centerMarker = new google.maps.Marker({
                position: {lat: {{ center.latitude }}, lng: {{ center.longitude }}},
                map: map,
                title: '{{ center.name|escapejs }}',
                icon: 'http://maps.google.com/mapfiles/ms/icons/green-dot.png'
            });

            {% for location in locations %}
                var marker = new google.maps.Marker({
                    position: {lat: {{ location.latitude|floatformat:6 }}, lng: {{ location.longitude|floatformat:6 }}},
                    map: map,
                    title: '{{ location.name|escapejs }}'
                });
                markers.push(marker);
            {% endfor %}

            var searchInput = document.getElementById('search-input');
            var searchBox = new google.maps.places.SearchBox(searchInput);

            map.addListener('bounds_changed', function() {
                searchBox.setBounds(map.getBounds());
            });
        }

        function searchLocation() {
            var searchInput = document.getElementById('search-input');
            var geocoder = new google.maps.Geocoder();
            geocoder.geocode({'address': searchInput.value}, function(results, status) {
                if (status === 'OK') {
                    map.setCenter(results[0].geometry.location);
                    calculateDistance(results[0].geometry.location);
                } else {
                    alert('Geocode was not successful for the following reason: ' + status);
                }
            });
        }

        function calculateDistance(destination) {
            var origin = new google.maps.LatLng({{ center.latitude }}, {{ center.longitude }});
            var request = {
                origin: origin,
                destination: destination,
                travelMode: 'DRIVING'
            };
            directionsService.route(request, function(result, status) {
                if (status == 'OK') {
                    directionsRenderer.setDirections(result);
                    var distance = result.routes[0].legs[0].distance.text;
                    var duration = result.routes[0].legs[0].duration.text;
                    document.getElementById('distance').innerHTML = 'Distance: ' + distance + '<br>Duration: ' + duration;
                }
            });
        }

        google.maps.event.addDomListener(window, 'load', initMap);
    </script>
</body>
</html>
