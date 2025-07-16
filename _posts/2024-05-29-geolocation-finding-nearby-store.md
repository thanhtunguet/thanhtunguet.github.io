---
title: "Finding Nearby Stores Using GPS Location in a Large Database with .NET Core"
date: 2024-05-29 00:01:00 +0700
categories: ["Software Development", "Backend"]
tags: [Programming, .NET Core]
pin: false
mermaid: true
---


### Introduction

In today's digital age, location-based services have become an integral part of many applications. Whether it's finding the nearest coffee shop, gas station, or grocery store, the ability to quickly and accurately locate nearby points of interest is crucial. This article explores how to efficiently find nearby stores using GPS location in a large database with thousands of stores using C# and .NET Core.

### The Problem

When dealing with a large database containing thousands of store locations, finding nearby stores based on a user's current GPS coordinates can be challenging. The primary challenges include:

1. **Performance:** Efficiently querying the database to return results within an acceptable time frame.
2. **Accuracy:** Ensuring the returned results are accurate and within the specified distance from the user's location.
3. **Scalability:** The solution must handle a growing database size without significant performance degradation.

### Solution Overview

To address these challenges, we can use a combination of geographical calculations and database indexing. The solution involves the following steps:

1. **Calculate a Bounding Box:** Determine a rectangular bounding box around the user's location that encompasses all points within the specified radius.
2. **Filter Points Using the Bounding Box:** Use the bounding box coordinates to filter out points that are definitely outside the desired radius.
3. **Calculate the Exact Distance:** For the points that fall within the bounding box, calculate the exact distance to ensure they are within the specified radius.
4. **Optimize with Indexing:** Use database indexing to improve the performance of the spatial queries.

### Step-by-Step Solution

#### Step 1: Calculate a Bounding Box

A bounding box is a rectangular area defined by minimum and maximum latitude and longitude values. This box helps to narrow down the number of points to consider before calculating the precise distance.

Here’s the function to calculate the bounding box:

```csharp
using System;

public class GeoUtils
{
    private const double EarthRadiusKm = 6371.0;

    public static (double MinLat, double MaxLat, double MinLong, double MaxLong) FindNearbyPoints(double currentLat, double currentLong, double radiusKm)
    {
        double minLat = currentLat - (radiusKm / EarthRadiusKm) * (180 / Math.PI);
        double maxLat = currentLat + (radiusKm / EarthRadiusKm) * (180 / Math.PI);
        double minLong = currentLong - (radiusKm / EarthRadiusKm) * (180 / Math.PI) / Math.Cos(currentLat * Math.PI / 180);
        double maxLong = currentLong + (radiusKm / EarthRadiusKm) * (180 / Math.PI) / Math.Cos(currentLat * Math.PI / 180);

        return (minLat, maxLat, minLong, maxLong);
    }
}
```

#### Step 2: Filter Points Using the Bounding Box

With the bounding box coordinates, we can filter out the points that are definitely outside the specified radius. This reduces the number of points for which we need to calculate the exact distance.

```csharp
using Microsoft.EntityFrameworkCore;
using System.Collections.Generic;
using System.Linq;

public class GpsPoint
{
    public double Latitude { get; set; }
    public double Longitude { get; set; }
}

public class GpsPointService
{
    private readonly YourDbContext _context;

    public GpsPointService(YourDbContext context)
    {
        _context = context;
    }

    public List<GpsPoint> GetPointsWithinRadius(double currentLat, double currentLong, double radiusKm)
    {
        var (minLat, maxLat, minLong, maxLong) = GeoUtils.FindNearbyPoints(currentLat, currentLong, radiusKm);

        var nearbyPoints = _context.GpsPoints
            .Where(p => p.Latitude >= minLat && p.Latitude <= maxLat && p.Longitude >= minLong && p.Longitude <= maxLong)
            .ToList();

        return nearbyPoints;
    }
}
```

#### Step 3: Calculate the Exact Distance

For the points that fall within the bounding box, calculate the exact distance using the Haversine formula to ensure they are within the specified radius.

```csharp
using System;

public static class DistanceUtils
{
    private const double EarthRadiusKm = 6371.0;

    public static double CalculateDistance(double lat1, double lon1, double lat2, double lon2)
    {
        double dLat = DegreesToRadians(lat2 - lat1);
        double dLon = DegreesToRadians(lon2 - lon1);

        double a = Math.Sin(dLat / 2) * Math.Sin(dLat / 2) +
                   Math.Cos(DegreesToRadians(lat1)) * Math.Cos(DegreesToRadians(lat2)) *
                   Math.Sin(dLon / 2) * Math.Sin(dLon / 2);
        double c = 2 * Math.Atan2(Math.Sqrt(a), Math.Sqrt(1 - a));
        return EarthRadiusKm * c;
    }

    private static double DegreesToRadians(double degrees)
    {
        return degrees * Math.PI / 180;
    }
}
```

#### Step 4: Filter by Exact Distance

Filter the points returned from the bounding box step by calculating the exact distance:

```csharp
public class GpsPointService
{
    private readonly YourDbContext _context;

    public GpsPointService(YourDbContext context)
    {
        _context = context;
    }

    public List<GpsPoint> GetPointsWithinRadius(double currentLat, double currentLong, double radiusKm)
    {
        var (minLat, maxLat, minLong, maxLong) = GeoUtils.FindNearbyPoints(currentLat, currentLong, radiusKm);

        var boundingBoxPoints = _context.GpsPoints
            .Where(p => p.Latitude >= minLat && p.Latitude <= maxLat && p.Longitude >= minLong && p.Longitude <= maxLong)
            .ToList();

        var nearbyPoints = boundingBoxPoints
            .Where(p => DistanceUtils.CalculateDistance(currentLat, currentLong, p.Latitude, p.Longitude) <= radiusKm)
            .ToList();

        return nearbyPoints;
    }
}
```

#### Step 5: Optimize with Indexing

To further optimize the performance, ensure that your database has indexes on the latitude and longitude columns. For example, in SQL Server, you can create a spatial index:

```sql
CREATE SPATIAL INDEX SIdx_GpsPoints_LatLong
ON GpsPoints (Latitude, Longitude);
```

### Conclusion

By calculating a bounding box, filtering points within that box, and then calculating the exact distances, we can efficiently find nearby stores in a large database. This approach combines geographical calculations with database optimization techniques to ensure both accuracy and performance. With proper indexing and efficient querying, your application can handle even large datasets seamlessly.

Implementing this solution in a .NET Core application ensures scalability and reliability, providing a solid foundation for location-based services.
