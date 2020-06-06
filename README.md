# iOS-MapKit-RuffSpots
iOS MapKit implementation for RuffSpots App

// MapKit view on the screen
// Pin constraints to the view (not safe area) at all 0
// Set delegate for view controller

//
// MapScreen.swift
// User-Location
//
// Created by Ritvik Ramdas on 5/26/20
// Copyright © 2020 Ritvik Ramdas. All rights reserved.
//

import UIKit
import MapKit
import CoreLocation

class MapScreen:UIViewController {

	@IBOutlet weak var mapView: MKMapView!
	@IBOutlet weak var addressLabel:UILabel!
	
	let locationManager = CLLocationManager()
	let regionMeters: Double = 10000
	var previousLocation: CLLocation?
	
	// Loads up the screen
	override func viewDidLoad() {
		super.viewDidLoad()
		checkLocationServices()
	}
	
	// Function to find best accuracy of user's location
	func setupLocationManager() {
		locationManager.delegate = self
		locationManager.desiredAccuracy = kCLLocationAccuracyBest
	}
	
	// Function to zoom into user's location
	func centerViewOnUserLocation() {
		if let location = locationManager.location?.coordinate {
			let region = MKCoordinateRegion.init(center: location, latitudinalMeters: regionInMeters,
				longitudinalMeters: regionInMeters)
			mapView.setRegion(region, animated: true)
		}
	}
	
	// Function to utilize user's device location settings (scope outside of Ruff Spots application)
	
	func checkLocationServices() {
		if CLLocationManager.locationServicesEnabled() {
			setupLocationManager()
			checkLocationAuthorization()
		} else {
			// Show alert letting the user know they have to turn this on
			let alertController = UIAlertController(Title: "Ruff Spots", message: "Please turn on your location services", preferredStyle: .alert)
			alertController.addAction(UIAlertAction(title: "Dismiss", style: .default))
			
			
			self.present(alertController, animated: true, completion: nil)
		
		}
	}
	
	// Function to check location authorization given to Ruff Spots by the user
	
	func checkLocationAuthorization() {
		switch CLLocationManager.authorizationStatus() {
		
		// authorized to get user's location only when app is open
		case .authorizedWhenInUse:
			startTackingUserLocation()
		case .denied:
			// Show alert instructing them how to turn on permissions
			break
		case .notDetermined:
			locationManager.requestWhenInUseAuthorization()
			break
		case .restricted:
			// Show an alert letting user know status
			break	
		// authorized to get user's location when app is running in the background
		case .authorizedAlways:
			break
		}
	}
	
	func startTackingUserLocation() {
		// Do Map Stuff
		// Shows blue dot of current location
		mapView.showsUserLocation = true
		centerViewOnUserLocation()
		locationManager.startUpdatingLocation()
		previousLocation = getCenterLocation(for: mapView)
	}
	
	func getCenterLocation(for mapView: MKMapView) -> CLLocation {
		let latitude = mapView.centerCoordinate.latitude
		let longitude = mapView.centerCoordinate.longitude
		
		return CLLocation(latitude: latitude, longitude: longitude)
	}
}


extension MapScreen: CLLocationManagerDelegate {

	// Function to update map's view of user's constantly updated location
	
	func locationManager(_ manager: CLLocationManagerDelegate, didUpdateLocations locations: [CLLocation]) {
		
		// Guards against no location
		guard let location = locations.last else { return }
		
		// Sets the center of the updated location of the user
		let center = CLLocationCoordinate2D(latitude: location.coordinate.latitude, longitude: location.coordinate.longitude)
		
		// Sets the width of the updated location of the user
		let region = MKCoordinateRegion.init(center: center, latitudinalMeters: regionInMeters, longitudinalMeters:
			regionInMeters)
			
		mapView.setRegion(region, animated: true)
		
	}
	
	func locationManager(_ manager: CLLocationManager, didChangeAuthorization status: CLAuthorizationStatus) {
		
		// Checks for any authorization changes
		checkLocationAuthorization()
	}
}

extension MapScreen: MKMapViewDelegate {
	func mapView(_ mapView: MKMapView, regionDidChangeAnimated animated: Bool) {
		let center = getCenterLocation(for: mapView)
		let geoCoder = CLGeocoder{}
		
		guard let previousLocation = self.previousLocation else { return }
		
		guard center.distance(from: previousLocation) > 50 else { return }
		self.previousLocation = center
		
		// Error checking
		geoCoder.reverseGeocodeLocation(center) { [weak self] (placemarks, error) in
			guard let self = self else { return }
			
			if let _ = error {
				//TODO: Show alert informing the user
				return
			}
			
			guard let placemark = placemarks?.first else {
				//TODO: Show alert informing the user
				return
			}
			
			let streetNumber = placemark.subThoroughfare ?? ""
			let streetName = placemark.thoroughfare ?? ""
			
			DispatchQueue.main.async {
				self.addressLabel.text = "\(streetNumber) \(streetName)"
			}
		}
	}
}
