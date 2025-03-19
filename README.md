import React, { useState, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";

const TripPlanner = () => {
  const [places, setPlaces] = useState([
    "Udupi Sri Krishna Temple",
    "Malpe Beach",
    "St. Mary’s Island",
    "Kapu Beach & Lighthouse",
    "Varanga Jain Temple",
    "Aagumbe Viewpoint",
    "Kudlu Theertha Falls",
    "Murudeshwar",
    "Honnavar Mangrove Boardwalk",
    "Apsarakonda Falls & Eco Beach",
    "Mirjan Fort",
    "Gokarna (Om Beach, Kudle Beach, Mahabaleshwar Temple)"
  ]);
  const [newPlace, setNewPlace] = useState("");
  const [map, setMap] = useState(null);
  const [directionsService, setDirectionsService] = useState(null);
  const [directionsRenderer, setDirectionsRenderer] = useState(null);

  useEffect(() => {
    if (window.google) {
      const mapInstance = new window.google.maps.Map(document.getElementById("map"), {
        center: { lat: 13.3409, lng: 74.7421 },
        zoom: 8,
      });
      setMap(mapInstance);
      setDirectionsService(new window.google.maps.DirectionsService());
      setDirectionsRenderer(new window.google.maps.DirectionsRenderer());
    }
  }, []);

  useEffect(() => {
    if (map && directionsRenderer) {
      directionsRenderer.setMap(map);
      if (places.length > 1) {
        calculateRoute();
      }
    }
  }, [map, directionsRenderer, places]);

  const calculateRoute = () => {
    if (!directionsService || places.length < 2) return;

    const waypoints = places.slice(1, -1).map(place => ({ location: place, stopover: true }));
    directionsService.route(
      {
        origin: places[0],
        destination: places[places.length - 1],
        waypoints: waypoints,
        optimizeWaypoints: true,
        travelMode: window.google.maps.TravelMode.DRIVING,
      },
      (result, status) => {
        if (status === "OK" && result) {
          directionsRenderer.setDirections(result);
        } else {
          console.error("Error fetching directions", status);
        }
      }
    );
  };

  const addPlace = () => {
    if (newPlace.trim() !== "") {
      setPlaces([...places, newPlace]);
      setNewPlace("");
    }
  };

  const removePlace = (index) => {
    const updatedPlaces = places.filter((_, i) => i !== index);
    setPlaces(updatedPlaces);
  };

  return (
    <div className="p-6 max-w-3xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">Trip Planner</h1>
      <div className="flex gap-2 mb-4">
        <Input
          value={newPlace}
          onChange={(e) => setNewPlace(e.target.value)}
          placeholder="Enter a place"
        />
        <Button onClick={addPlace}>Add Place</Button>
      </div>
      <div className="space-y-3">
        {places.map((place, index) => (
          <Card key={index} className="p-4 flex justify-between items-center">
            <CardContent>{place}</CardContent>
            <Button variant="destructive" onClick={() => removePlace(index)}>
              Remove
            </Button>
          </Card>
        ))}
      </div>
      <div id="map" className="w-full h-96 mt-6"></div>
    </div>
  );
};

export default TripPlanner;
