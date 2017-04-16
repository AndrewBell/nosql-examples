# Get all contents
`curl -v -X GET localhost:5984/catalog/_design/contents/_view/all-contents | python -mjson.tool`

# Get all contents of entities, including docs
`curl -v -X GET localhost:5984/catalog/_design/contents/_view/all-contents?include_docs=true | python -mjson.tool`

# Get content keys for specific entity
`curl -X GET localhost:5984/catalog/_design/contents/_view/all-contents?include_docs=true -G --data-urlencode start_key='["735174c9bbfec0115fa1e65e63001edc",""]' --data-urlencode end_key='["735174c9bbfec0115fa1e65e63001edc",{}]' | python -mjson.tool`

# Get all contents for a specific entity
`curl -X GET localhost:5984/catalog/_design/contents/_view/all-contents-and-amounts?include_docs=true -G --data-urlencode start_key='["735174c9bbfec0115fa1e65e63001edc",""]' --data-urlencode end_key='["735174c9bbfec0115fa1e65e63001edc",{}]' | python -mjson.tool`
