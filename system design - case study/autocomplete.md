
***
### Clarifying Questions
* support rich media (images, videos) results? - yes (thumbnail image)
* what devices supported? - mobile, tablet, desktop
* fuzzy search supported (handling typos and still show correct results)? - yes
* can be used for news, stocks, social media search.
### Requirements
* ***Frontend UI** - Customizable Reusable Component:
	* Input Field box
	* Search Results

```
// Autocomplete Component

<AutoComplete {...props}/>

Props:
* Number of Results
* Server API URL
* event listener handlers
	- input   - select
	- focus   - blur
	- change 
* custom rendering/styling (theming options, classNames, render functions)
```
* *render functions*
```
render() {
 const {
 // ...other props ...
 renderInput,
 renderResults,
 renderResultItem,
 // ...more render functions...
 } = this.props;

 return (
	<div className="autocomplete-container">
	{renderInput({
		// ... pass input-related props ...
	})}
	{isOpen && renderResults({
		results: results
		// ... pass other result-related props ...
	})}
	</div>
 )
}
```

* ***Backend - Server Requirements**
	* *query*: `api.endpoint/autocomplete?query=QUERY&limit=10&pindex=0`
		* endpoint url
		* query parameters (QUERY, LIMIT, Page Index)
* ***Network Requirements***
	* options:
		1. Timestamp each request - *display latest request*.
		2. save responses in Map[] - *with search query string as KEY*., **Preferred alternative incase of caching system.**
	* *Network Failure:* if server doesn't respond - alert notification about failure and option to retry. or auto-retry in the back.  Still, if issue persists - implementation of exponential back-off strategy to prevent congesting the network.
		* In case of no network connection, can only query the cache (for stale responses) and restrict server calls to save compute (CPU).
### Rendering - Frontend Performance
* **Virtualized List** / **Windowing** - search results are displayed a  virtualized list. To handle computational load, nodes(each item space in the list) is recycled.
* **pagination** - search results.
* fast loading speed - use cached results, debounce text entry.
* purge cache - on idle browser, entries exceeds threshold., smart/partial purge(LRU)
***
![[Pasted image 20240403191404.png]]
***
### Component Parameters
* **minimum query length** (min 3 characters)
* **input debounce duration**
* **Server API timeout duration**
* **caching options**
	* *initial results*
	* *results source* (caching configuration)
	* cache duration
### Cache Design
* ***Hashmap / Dictionary*:** with search term as `KEY`. *Fast and cheap O(1) lookup time.* But, will lead to huge number of *duplicate results.* Ideal for Short-lived Session. (one-time use)
* ***Flatlist***: will lead to frontend performance as the list grows. not ideal to store ordered ranking of results. Ideal for Long-lived session with history from previous sessions.
* Hybrid Approach that combines the above two. - **Normalized Map of Results** - structure the cache as a database. Each result entry is considered as a row with unique id. w/ a cache lookup. *Fast lookup with no duplication.*
### Cache Strategy
* avoid stale data. 
* when to purge cache?
	* if results are slow to change - **less often** (i.e. - Google)
	* if results change fast - **more often** (i.e. - Facebook)
	* Real-time use case - **instantaneous / omit cache** (i.e.- Stock ticker)
* *data source*: a. network only, b. network and cache, c. cache only.
***
### UI/UX Improvements
* **UI states**:
	* loading state, skeleton loaders.
	* error state, error boundary.
	* no network
* handle long result texts - *truncation*
* **Mobile Consideration**:
	* results item large enough to be tapped on
	* show mobile-friendly length of results - *better handled in user preferences/configuration*
	* **mobile attributes** - *auto capitalize, auto complete, auto correct, spellcheck off (w/ fuzzy search)*
* **keyboard interaction (`cmdk`) / hotkey**:
	* *global shortcut to autofocus*.
	* up/down - navigate results.
	* "enter" triggers search.
	* "escape" dismisses results.
* **Accessibility** - considers disabilities. 
	* Blind Users - Keyboard Navigation & Screen-Readers (directives to audio).
	* Semantic HTML - WAI-ARIA