# Spatial-Databases-Project  
H.Dip.-Data-Analytics  
SQL Code used with QGIS  

# -- Deleting the six northern counties FROM the counties table as they are not part of the study area. 
BEGIN;  
SELECT * FROM counties WHERE name_tag = 'Londonderry' OR name_tag = 'Antrim' OR name_tag = 'Fermanagh' OR name_tag = 'Tyrone' OR name_tag = 'Armagh' OR name_tag = 'Down';  
DELETE  FROM counties WHERE name_tag = 'Londonderry' OR name_tag = 'Antrim' OR name_tag = 'Fermanagh' OR name_tag = 'Tyrone' OR name_tag = 'Armagh' OR name_tag = 'Down';  
COMMIT;  
ROLLBACK;  
  
# -- Removing secondary roads FROM the OSM road dataset - has already been partially filtered using QGIS. 
BEGIN;  
SELECT * FROM gisosmroads WHERE fclass = 'secondary' OR fclass = 'secondary_link';  
DELETE  FROM gisosmroads WHERE fclass = 'secondary' OR fclass = 'secondary_link';  
COMMIT;  
ROLLBACK;  
    
# -- Number of primary schools in each county  
ALTER TABLE counties ADD COLUMN NumPrimarySchools INTEGER DEFAULT 0;  
With PolygonQuery as (  
    SELECT C.name_tag,count(*) as NumPrimarySchoolsInCounty   
	FROM counties as C, primaryschools as P  
	WHERE ST_CONTAINS(C.wkb_geometry,P.wkb_geometry)  
	GROUP BY C.name_tag)  
UPDATE counties  
SET NumPrimarySchools = PolygonQuery.NumPrimarySchoolsInCounty    
FROM PolygonQuery   
WHERE counties.name_tag = PolygonQuery.name_tag;  


  
# -- Number of post primary schools in each county  
ALTER TABLE counties ADD COLUMN NumPostPrimarySchools INTEGER DEFAULT 0;  
  
With PolygonQuery as (  
    SELECT C.name_tag,count(*) as NumPostPrimarySchoolsInCounty   
	FROM counties as C, postprimaryschools as P  
	WHERE ST_CONTAINS(C.wkb_geometry,P.wkb_geometry)  
	GROUP BY C.name_tag)  
UPDATE counties  
SET NumPostPrimarySchools = PolygonQuery.NumPostPrimarySchoolsInCounty    
FROM PolygonQuery   
WHERE counties.name_tag = PolygonQuery.name_tag;  

  
# -- View of primary schools  
DROP VIEW IF EXISTS PrimarySchoolsView;  
CREATE OR REPLACE VIEW PrimarySchoolsView as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, C.name_tag as CountyName, P.wkb_geometry as schgeom  
FROM counties as C, primaryschools as P  
WHERE ST_Contains(C.wkb_geometry,P.wkb_geometry);    
  
SELECT count(*) FROM p210004.PrimarySchoolsView  
  
# -- View of post primary schools  
DROP VIEW IF EXISTS PostPrimarySchoolsView;  
CREATE OR REPLACE VIEW PostPrimarySchoolsView as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, C.name_tag as CountyName, P.wkb_geometry as schgeom  
FROM counties as C, postprimaryschools as P  
WHERE ST_Contains(C.wkb_geometry,P.wkb_geometry);   
  
SELECT count(*) FROM p210004.PostPrimarySchoolsView  
  
# --Counting schools in Dublin  
# --Primary  
SELECT count(*)  
FROM counties as C, primaryschools as P  
WHERE ST_Contains(C.wkb_geometry,P.wkb_geometry) AND C.name_tag = 'Dublin';    
  
# --Post Primary  
SELECT count(*)  
FROM counties as C, postprimaryschools as P  
WHERE ST_Contains(C.wkb_geometry,P.wkb_geometry) AND C.name_tag = 'Dublin';  
  
# -- Dublin schools within NO2 limits  
# -- Primary  
DROP VIEW IF EXISTS NO2PrimarySchoolsView;  
CREATE OR REPLACE VIEW NO2PrimarySchoolsView as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, N.range as NO2Range, P.wkb_geometry as schgeom  
FROM airno2 as N, primaryschools as P  
WHERE ST_Contains(N.wkb_geometry,P.wkb_geometry)  
AND N.min_ >= 40;    
  
SELECT count(*) FROM p210004.NO2PrimarySchoolsView  
  
# --Post Primary  
DROP VIEW IF EXISTS NO2PostPrimarySchoolsView;  
CREATE OR REPLACE VIEW NO2PostPrimarySchoolsView as   
SELECT  P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, N.range as NO2Range, P.wkb_geometry as schgeom  
FROM airno2 as N, postprimaryschools as P  
WHERE ST_Contains(N.wkb_geometry,P.wkb_geometry)  
AND N.min_ >= 40;    
  
SELECT count(*) FROM p210004.NO2PostPrimarySchoolsView  
  
# -- Dublin schools within PM10 limits  
# -- Primary  
DROP VIEW IF EXISTS PM10PrimarySchoolsView;  
CREATE OR REPLACE VIEW PM10PrimarySchoolsView as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, N.range as PM10Range, P.wkb_geometry as schgeom  
FROM airpm10 as N, primaryschools as P  
WHERE ST_Contains(N.wkb_geometry,P.wkb_geometry)  
AND N.min_ > 0;    
  
SELECT count(*) FROM p210004.PM10PrimarySchoolsView  
  
# --Post Primary  
DROP VIEW IF EXISTS PM10PostPrimarySchoolsView;  
CREATE OR REPLACE VIEW PM10PostPrimarySchoolsView as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, N.range as PM10Range, P.wkb_geometry as schgeom  
FROM airpm10 as N, postprimaryschools as P  
WHERE ST_Contains(N.wkb_geometry,P.wkb_geometry)  
AND N.min_ > 0;    
  
SELECT count(*) FROM p210004.PM10PostPrimarySchoolsView  

  
# -- Dublin schools within PM2.5 limits  
# -- Primary  
DROP VIEW IF EXISTS PM25PrimarySchoolsView;  
CREATE OR REPLACE VIEW PM25PrimarySchoolsView as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, N.range as PM25Range, P.wkb_geometry as schgeom  
FROM airpm25 as N, primaryschools as P  
WHERE ST_Contains(N.wkb_geometry,P.wkb_geometry)  
AND N.min_ > 0;    
  
SELECT count(*) FROM p210004.PM25PrimarySchoolsView  
  
# --Post Primary  
DROP VIEW IF EXISTS PM25PostPrimarySchoolsView;  
CREATE OR REPLACE VIEW PM25PostPrimarySchoolsView as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, N.range as PM25Range, P.wkb_geometry as schgeom  
FROM airpm25 as N, postprimaryschools as P  
WHERE ST_Contains(N.wkb_geometry,P.wkb_geometry)  
AND N.min_ > 0;    
  
SELECT count(*) FROM p210004.PM25PostPrimarySchoolsView  
  
# -- All schools within 100m of a main road  
# --Primary  
DROP TABLE IF EXISTS Road100mPrimarySchools CASCADE;  
CREATE  TABLE Road100mPrimarySchools as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, P.wkb_geometry as schgeom, St_Transform(N.wkb_geometry, 32629) <-> St_Transform(P.wkb_geometry, 32629) as DISTANCE  
FROM gisosmroads as N, primaryschools as P  
WHERE ST_DWithin(St_Transform(N.wkb_geometry, 32629),St_Transform(P.wkb_geometry, 32629),100)  
ORDER BY schoolid DESC;  
  

DROP TABLE IF EXISTS Road100mPrimarySchoolsNoDup;  
CREATE TABLE Road100mPrimarySchoolsNoDup as   
SELECT DISTINCT ON (SchoolID) Schoolname, Town, schgeom  
FROM Road100mPrimarySchools  
ORDER BY SchoolID;  
  
SELECT count(*) FROM p210004.Road100mPrimarySchools  
SELECT count(*) FROM p210004.Road100mPrimarySchoolsNoDup  
  

# --Post Primary  
DROP TABLE IF EXISTS Road100mPostPrimarySchools CASCADE;  
CREATE  TABLE Road100mPostPrimarySchools as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, P.wkb_geometry as schgeom, St_Transform(N.wkb_geometry, 32629) <-> St_Transform(P.wkb_geometry, 32629) as DISTANCE  
FROM gisosmroads as N, postprimaryschools as P  
WHERE ST_DWithin(St_Transform(N.wkb_geometry, 32629),St_Transform(P.wkb_geometry, 32629),100)  
ORDER BY schoolid DESC;  
  
DROP TABLE IF EXISTS Road100mPostPrimarySchoolsNoDup;  
CREATE TABLE Road100mPostPrimarySchoolsNoDup as   
SELECT DISTINCT ON (SchoolID) Schoolname, Town, schgeom  
FROM Road100mPostPrimarySchools  
ORDER BY SchoolID;  
  
SELECT count(*) FROM p210004.Road100mPostPrimarySchools  
SELECT count(*) FROM p210004.Road100mPostPrimarySchoolsNoDup  



# -- All points within 100m of traffic lights  
# --Primary  
DROP TABLE IF EXISTS Traffic100mPrimarySchools CASCADE;  
CREATE  TABLE Traffic100mPrimarySchools as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, P.wkb_geometry as schgeom, St_Transform(N.wkb_geometry, 32629) <-> St_Transform(P.wkb_geometry, 32629) as DISTANCE  
FROM gisosmTraffic as N, primaryschools as P  
WHERE ST_DWithin(St_Transform(N.wkb_geometry, 32629),St_Transform(P.wkb_geometry, 32629),100)  
ORDER BY schoolid DESC;  
  
DROP TABLE IF EXISTS Traffic100mPrimarySchoolsNoDup;  
CREATE TABLE Traffic100mPrimarySchoolsNoDup as   
SELECT DISTINCT ON (SchoolID) Schoolname, Town, schgeom  
FROM Traffic100mPrimarySchools  
ORDER BY SchoolID;  
  
SELECT count(*) FROM p210004.Traffic100mPrimarySchools  
SELECT count(*) FROM p210004.Traffic100mPrimarySchoolsNoDup  
  
# --Post Primary  
DROP TABLE IF EXISTS Traffic100mPostPrimarySchools CASCADE;  
CREATE  TABLE Traffic100mPostPrimarySchools as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, P.wkb_geometry as schgeom, St_Transform(N.wkb_geometry, 32629) <->   St_Transform(P.wkb_geometry, 32629) as DISTANCE  
FROM gisosmTraffic as N, postprimaryschools as P  
WHERE ST_DWithin(St_Transform(N.wkb_geometry, 32629),St_Transform(P.wkb_geometry, 32629),100)  
ORDER BY schoolid DESC;  
  
DROP TABLE IF EXISTS Traffic100mPostPrimarySchoolsNoDup;  
CREATE TABLE Traffic100mPostPrimarySchoolsNoDup as   
SELECT DISTINCT ON (SchoolID) Schoolname, Town, schgeom  
FROM Traffic100mPostPrimarySchools  
ORDER BY SchoolID;  
  
SELECT count(*) FROM p210004.Traffic100mPostPrimarySchools  
SELECT count(*) FROM p210004.Traffic100mPostPrimarySchoolsNoDup  

  
# -- All schools within 100m of a main train line  
# --Primary  
DROP TABLE IF EXISTS Rail100mPrimarySchools CASCADE;  
CREATE TABLE Rail100mPrimarySchools as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, P.wkb_geometry as schgeom, St_Transform(N.wkb_geometry, 32629) <-> St_Transform(P.wkb_geometry, 32629) as DISTANCE  
FROM gisosmrailways as N, primaryschools as P  
WHERE ST_DWithin(St_Transform(N.wkb_geometry, 32629),St_Transform(P.wkb_geometry, 32629),100)  
ORDER BY schoolid DESC;  
  
DROP TABLE IF EXISTS Rail100mPrimarySchoolsNoDup;  
CREATE TABLE Rail100mPrimarySchoolsNoDup as   
SELECT DISTINCT ON (SchoolID) Schoolname, Town, schgeom  
FROM Rail100mPrimarySchools  
ORDER BY SchoolID;  
  
SELECT count(*) FROM p210004.Rail100mPrimarySchools  
SELECT count(*) FROM p210004.Rail100mPrimarySchoolsNoDup  


  
# --Post Primary  
DROP TABLE IF EXISTS Rail100mPostPrimarySchools CASCADE;  
CREATE TABLE Rail100mPostPrimarySchools as   
SELECT P.ogc_fid as SchoolID, P.off_name as Schoolname, P.add_1 as Town, P.wkb_geometry as schgeom, St_Transform(N.wkb_geometry, 32629) <->   St_Transform(P.wkb_geometry, 32629) as DISTANCE  
FROM gisosmrailways as N, postprimaryschools as P  
WHERE ST_DWithin(St_Transform(N.wkb_geometry, 32629),St_Transform(P.wkb_geometry, 32629),100)  
ORDER BY schoolid DESC;  
  
DROP TABLE IF EXISTS Rail100mPostPrimarySchoolsNoDup;  
CREATE TABLE Rail100mPostPrimarySchoolsNoDup as   
SELECT DISTINCT ON (SchoolID) Schoolname, Town, schgeom  
FROM Rail100mPostPrimarySchools  
ORDER BY SchoolID;  
  
SELECT count(*) FROM p210004.Rail100mPostPrimarySchools  
SELECT count(*) FROM p210004.Rail100mPostPrimarySchoolsNoDup  



---------------------------------------------------------------------  
# -- Merging all the tables created into one table for primary schools, and one table for post primary schools, then removing all duplicates  

# -- Primary Schools  
DROP TABLE IF EXISTS AllPrimarySchools CASCADE;  
CREATE TABLE AllPrimarySchools as   
SELECT * FROM Road100mPrimarySchools  
UNION SELECT * FROM Traffic100mPrimarySchools  
UNION SELECT * FROM Rail100mPrimarySchools  
  
DROP TABLE IF EXISTS AllPrimarySchoolsNoDup;  
CREATE TABLE AllPrimarySchoolsNoDup as   
SELECT DISTINCT ON (SchoolID) Schoolname, Town, schgeom  
FROM AllPrimarySchools  
ORDER BY SchoolID;  
  
SELECT count (*)  FROM AllPrimarySchools  
SELECT count (*) FROM AllPrimarySchoolsNoDup  


# --Post Primary Schools  
DROP TABLE IF EXISTS AllPostPrimarySchools CASCADE;  
CREATE TABLE AllPostPrimarySchools as   
SELECT * FROM Road100mPostPrimarySchools  
UNION SELECT * FROM Traffic100mPostPrimarySchools  
UNION SELECT * FROM Rail100mPostPrimarySchools  
  
DROP TABLE IF EXISTS AllPostPrimarySchoolsNoDup;  
CREATE TABLE AllPostPrimarySchoolsNoDup as   
SELECT DISTINCT ON (SchoolID) Schoolname, Town, schgeom  
FROM AllPostPrimarySchools  
ORDER BY SchoolID;  
  
SELECT count (*)  FROM AllPostPrimarySchools  
SELECT count (*) FROM AllPostPrimarySchoolsNoDup  





