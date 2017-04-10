# bayesLopod
###### Bayes inference of Landscape Occupancy from Presence-Only Data

Natural history museums and herbaria collectively hold hundreds of millions of zoological, botanical, and paleontological specimens. These collections serve as the foundation for understanding the distribution of life on Earth and the basis for addressing loss of biodiversity, emerging diseases, and other pressing global problems as well as important question in ecology and evolution. One of the short comings of these kind of data is that the lack of evidence of the presence of a species in a certain region does not mean the species is truly absent there. Likewise, specimens are often misidentified, and therefore the report of a species in a locality is not always evidence that a viable population occurs there. The goal of this project is to develop a method which could be used to estimate the probability of presence of a species in a certain study region based on certain sampling effort and presence reports.

***

## bayesLOPOD package structure (To-Do list)
(* necessary for version 1.0):

### Input
###### All these functions will create a LopodData object which will be a list of the original geographic structure of the data (Raster or Shape) and a list of data ready for Stan. In all Elements at least 1 element must have Y > 1 and in all of them N > Y.

- [X] rasterLopodData(rasterN, rasterY, extSample, extDetection) *

- [ ] xyLopodData(xyN, xyY,nCells, extSample, extDetection)

- [ ] shapeLopodData(ShapeFile, fieldN, fieldY)

- [ ] dfLopodData(df, fieldN, fieldY, custW)

### Model (and Stan Scripts)

###### All Stan files will be called from the same function depending on the attributes. The result is a LopodModel object which is a list of the settings used to run the model,  Stan model output and the LopodData geo object.

- [X] modelLopod(LopodData, varP = F, q =  NULL, pmin = 0, CAR = F, nChains = 4, warmup = 2000, sampling = 1000)*

``` {r}

if (q<0  & q>1){
 message("q must be NULL or between 0 and 1")
}

if (pmin<0  & pmin>1){
 message(pmin must be between 0 and 1")
}
```

#### Most parameterized stanData

``` {r}
stanData = list( nSampledCells = length(LopodData@geoInfo$sampledId$cellRaster),
                         sampledId = LopodData@geoInfo$sampledId$cellStan,
                         nNotSampled = length(LopodData@geoInfo$notSampledId$cellStan),
                         notSampledId = LopodData@geoInfo$notSampledId$cellStan,
                         n = length(LopodData@geoInfo$sampledId$cellRaster)+length(LopodData@geoInfo$notSampledId$cellStan),
                         W_n = dim(LopodData@geoInfo$W_sparse)[1],
                         W_sparse = LopodData@geoInfo$W_sparse,
                         D_sparse = LopodData@geoInfo$D_sparse,
                         lambda = LopodData@geoInfo$lambda_sparse,
                         N = N,
                         y = y,
                         minP = pmin,
                         q = q

        )
      ```


#### Stan Files

``` {r}
if (CAR == F) {

  if (varP == F){

    if (is.null(q)==T){
```
###### Global p and q estimated. Psy for each sampling unit.

- [X] psyipq.stan *


``` {r}
    }

    if (is.null(q)==F) {  

        pmin = q
```
###### Global p estimated assuming p is larger than the given q (which can be 0, in which case there are no false detections). Psy estimated for each sampling unit.

- [X] psyip.stan *

``` {r}
    }
  }

  if (varP == T){

    if (is.null(q)==T){
```
###### Global q estimated. Psy and P for each sampling unit.

- [X] psyipiq.stan *


``` {r}
    }

    if (is.null(q)==F) {  

        pmin = q
```
###### Psy and p estimated for each sampling unit assuming p is larger than the given q (which can be 0, in which case there are no false detections).

- [X] psyipi.stan *

``` {r}
    }
  }
}

if (CAR == T) {
  if(is.null(LopodData$stanData$w)==T){
    message("Cannot perform CAR analysis without adjacency matrix")
  } else {

  if (varP == F){

    if (is.null(q)==T){
```
###### Global p and q estimated. Psy for each sampling unit. Psy is spatially autocorrelated.

- [X] psyipq_CAR.stan *


``` {r}
    }

    if (is.null(q)==F) {  

        pmin = q
```
###### Global p estimated assuming p is larger than the given q (which can be 0, in which case there are no false detections). Psy estimated for each sampling unit. Psy is spatially autocorrelated.

- [X] psyip_CAR.stan *

``` {r}
    }
  }

  if (varP == T){

    if (is.null(q)==T){
```
###### Global q estimated. Psy and P for each sampling unit. Psy is spatially autocorrelated.

- [X] psyipiq_CAR.stan *


``` {r}
    }

    if (is.null(q)==F) {  

        pmin = q
```
###### Psy and p estimated for each sampling unit assuming p is larger than the given q (which can be 0, in which case there are no false detections). Psy is spatially autocorrelated.

- [X] psyipi_CAR.stan *

``` {r}
    }
  }
  }
}
```
### Output

###### Summary statistic of the output of the Stan models as well as occupancy models / CAR parameters depending on the model used.  

- [ ] lopodSummary(LopodModel) *

###### Maps of the parameters estimated for each sampling unit. For shapes these will be added to the attribute table

- [ ] lopodRaster(LopodModel, par="psy", value="median") *

- [ ] lopodShape(shapefle, LopodModel, par="psy", value="median", fieldname = NULL)

###### Stacks/Shapefiles with the maps of the parameters estimated for each sampling unit per iteration.

- [ ] lopodRasterDist(LopodModel, par="psy", value="median", nIter = 100)

- [ ] lopodShapeDist(shapefle, LopodModel, par="psy", value="median", fieldname = NULL, nIter = 100)

###### SpatialDataframe of XY with presence/absence or presence/background for other SDM programs based on a threshold.

- [ ] lopodPresAbs(LopodModel, par="psy", value="median", thresh=0.05)

- [ ] lopodPresBG(LopodModel, par="psy", value="median", thresh=0.05)


###### SpatialDataframe of XY with presence/absence or presence/background for other SDM programs based on a threshold.

- [ ] lopodPresAbsDist(LopodModel, par="psy", nIter = 100)

- [ ] lopodPresBGDist(LopodModel, par="psy", nIter = 100)
