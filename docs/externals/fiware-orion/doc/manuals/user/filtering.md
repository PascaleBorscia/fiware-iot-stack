# Filtering results

Orion Context Broker implements several filters
that can be used to filter the results in NGSI10 query operations. These
filters are typically used with [queryContext with patterns](walkthrough_apiv1.md#query-context-operation) or [the convenience operation to get all entities](walkthrough_apiv1.md#getting-all-entities).

As a general rule, filters used in standard operation use a scope
element:

 
      (curl localhost:1026/v1/queryContext -s -S --header 'Content-Type: application/json' --header 'Accept: application/json' -d @- | python -mjson.tool) <<EOF
      {
	  "entities": [
	      {
		  "type": "myEntityType",
		  "isPattern": "true",
		  "id": ".*"
	      }
	  ],
	  "restriction": {
	      "scopes": [
		  {
		      "type": "FIWARE::Filter::foobar",
		      "value": ""
		  }
	      ]
	  }
      }
      EOF                                                                                                                 

while filters in convenience operations are included as parameters in
the URL:

    curl localhost:1026/v1/contextEntities?filter=value -s -S --header 'Content-Type: application/json' --header 'Accept: application/json' | python -mjson.tool

Filters are cumulative. In other words, you can use several scopes in
the same restriction (in the case of standard operations) or several URL
argument separated by '&' in order to specify several filters. The
result is a logic "and" between all of them.

## Existence type filter

The scope correspoding to this type is "FIWARE::Filter::Existence". 

      ...                                                           
      {
	  "restriction": {
	      "scopes": [
		  {
		      "type": "FIWARE::Filter::Existence",
		      "value": "entity::type"
		  }
	      ]
	  }
      }
      ...                                                           
  
The URL parameter corresponding to this filter is 'exist'.

    curl localhost:1026/v1/contextEntities?exist=entity::type ...

In the current version, the only parameter than can be checked for
existence is the entity type, corresponding to "entity::type".

## No-Existence type filter

The scope corresponding to this type is "FIWARE::Filter::Not::Existence".

      ...                                                                
      {
	  "restriction": {
	      "scopes": [
		  {
		      "type": "FIWARE::Filter::Not::Existence",
		      "value": "entity::type"
		  }
	      ]
	  }
      }
      ...                                                                
  
The URL parameter corresponding to this filter is '!exist'.

    curl localhost:1026/v1/contextEntities?!exist=entity::type ...

In the current version, the only parameter than can be checked for
no-existence is the entity type, corresponding to "entity::type". Note
that this is the only way of selecting an "entity without type" (given
that queries without type resolve to "any type", as explained in the
[following section](empty_types.md#using-empty-types)).

## Entity type filter

There is no scope corresponding to this filter, given that you can use
the usual entity type:

  --------------------------------------------------------------------------------------
  JSON
  --------------------------------------------------------------------------------------
      ...                                            
      {
	  "type": "Room",
	  "isPattern": "...",
	  "id": "..."
      }
      ...
  --------------------------------------------------------------------------------------

The URL parameter corresponding to this filter is 'entity::type'.

    curl localhost:1026/v1/contextEntities?entity::type=Room ...

## Geo-location filter

The scope corresponding to this type is "FIWARE::Location". It is
described in detail in [the following section](geolocation.md#geo-located-queries).

In the current version of Orion, there is no equivalent convenience
operation filter.
