# Pathfinder's Codebase

Welcome to Pathfinder's codebase! This repository serves primarily as a wiki for various topics of discussion between internal team members, our project manager and our client.

## About

We are a team of students from Rose-Hulman Institute of Technology. As our senior capstone project for the Computer Science department we have created Pathfinder, a "route-optimization-platform-as-a-service" (abbreviation pending). Check it out at [https://thepathfinder.xyz](https://thepathfinder.xyz).

- Adam Michael <adam@ajmichael.net>
- David Robinson <robinsdm@rose-hulman.edu>
- Dan Hanson <hansondg@rose-hulman.edu>

All of our code is open source, [MIT licensed](https://raw.githubusercontent.com/CSSE497/pathfinder/master/LICENSE) and hosted within this GitHub organization.

## List of Repositories

Over the course of Pathfinder's development, our codebase has gotten fairly large. It is split into several repositories to help us manage our continued development process.

### [`pathfinder-android`](https://github.com/csse497/pathfinder-android)
The official Android SDK for using Pathfinder within an Android app. The code is also available on Maven Central.

### [`pathfinder-ios`](https://github.com/csse497/pathfinder-ios)
The official iOS SDK for using Pathfinder within an iOS app. The code is also available on Cocoapods.

### [`pathfinder.js`](https://github.com/csse497/pathfinder.js)
The official Javascript library for using Pathfinder within a web app. The code is also available on Bower.

### [`pathfinder-webserver`](https://github.com/csse497/pathfinder-webserver)
This is a Play application that hosts `https://thepathfinder.xyz` including all assets and an interactive dashboard that allows developers to interact with their Pathfinder applications via Pathfinder.js.

### [`pathfinder-server`](https://github.com/csse497/pathfinder-server)
This is a Play application that exposes that Pathfinder websocket API at `https://api.thepathfinder.xyz`.

### [`pathfinder-routing`](https://github.com/csse497/pathfinder-routing)
This repository contains four webservers. Three compute route optimizations and the fourth is a "master router" that selects the optimal route from a server farm made up of instances of the worker servers.

### [`simulatedannealing`](https://github.com/csse497/pathfinder-simulatedannealing)
This is a Java library for performing simulated annealing. It is used by the `pathfinder-routing`

### [`authentication-server`](https://github.com/csse497/authentication-server)
This is a NodeJs application that powers Pathfinder-hosted authentication for applications at `https://auth.thepathfinder.xyz`.

### [`pathfinder-db`](https://github.com/csse497/pathfinder-db)
This repository holds our database evolution scripts and various other tools that we used to manage our database.

### [`ChimneySwap`](https://github.com/csse497/ChimneySwap)
This is the backend for a sample application that uses Pathfinder to compute routes. There are corresponding Android and iOS sample applications to go with the backend in the `examples` directory of the Android and iOS SDKs.

### [`simulator`](https://github.com/csse497/simulator)
This is a JavaFX desktop application that simulates a driver for the ChimneySwap sample app. We primarily use it for demos because it is easier than recruiting a friend to drive around moving chimneys.
