## Drop sgb128.graphml to the import directory for your database
Mine was: C:\\Users\\jesus\\.Neo4jDesktop\\relate-data\\dbmss\\dbms-1ea5ef78-5628-47a1-a31a-794be174cf4d\\import\\sgb128.graphml

## Run the apoc import
`call apoc.import.graphml('sgb128.graphml', {readLabels: true})`

## Run a query which adds labels to the nodes
`MATCH(n)
call apoc.create.addLabels(id(n), [n.label])
yield node
remove node.label
return node`

## Roughly convert the x,y to lat and lon
`match (n)
set n.longitude = toFloat(n.x_pos)/55
set n.latitude = toFloat(n.y_pos)/69
return n`

## Add nautical miles drive distance to routes
`MATCH (n)-[r:ROUTE]->()
set r.nm_drive = toFloat(r.drive)*0.868976
RETURN r`

## Run the A* Algo
`match(start:City {name:'Winnipeg, MB'}), (end:City {name: 'West Palm Beach, FL'})
call gds.alpha.shortestPath.astar.stream({
	nodeProjection: {
    	City: {
        	properties: ['longitude', 'latitude']
        }
    },
    relationshipProjection: {
    	ROUTE: {
        	type: 'ROUTE',
            orientation: 'UNDIRECTED',
            properties: 'nm_drive'
        }
    },
    startNode: start,
    endNode: end,
    propertyKeyLat: 'latitude',
    propertyKeyLon: 'longitude'
})
yield nodeId, cost
return gds.util.asNode(nodeId).name as City, cost`

## Run these two queries to delete your whole database:
`match (a) -[r] -> () delete a,r`

`match (a) delete a`