---
title: Crop mapping forms
layout: default
permalink: '/crop-mapping-form'
---
## Crop Type survey form

The crop type survey form was developed to take the crop in the plot, along with the location and polyogonal shape data of the plot. 

<a href="./survey_form.xlsx" download> Download the Crop type survey form </a>

### Plot selection : 

The form was designed such that the plot data collected fulfilled certain requirement.

1.  **Plot_size**: Is this plot at least 20 meters by 20 meters in size?

        Type: select one (Yes/No)

    It is important to make sure the plot is at least 20 * 20 meteres to ensure we get good data.
2.  **Nearby_plot**:	Is this plot touching any previouly sampled plots?

        Type: select one (Yes/No)

    Two sampled plots should not be touching one another, as this might cause overlapping of the polygonal data, which gives bad data.
3. **adm_1**: Select the highest adminstrative division in the country

        Type : Select one
4. **adm_2** : Select the second highest adminstrative division in the country
    
        Type: Select one 

### Crop selection

1. **crop** : Select the crop currently being grown/Landuse in this plot

        Type : Select one (from the crop list)

### Geo-location

1. **image_widget** : Take an image of the field

        Type: The max-pixels can be defined (eg: 1024)
2. **PlotGPS**	GPS from the center of the experiment plot

        Type : Geo-point of the the centre of the plot 
    
    The accuracy can be set to less than desired meters (e.g. 10 meters). By default the value is 100m
3. **shape** : Record shape by tapping on each corner of the experiment plot that you see on the map
4. **shape_area** : The shape area of the polygon is calculated by the system
5. **rounded_shape_area** : The calculated shape area is rounded using the round function. 


