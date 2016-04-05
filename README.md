Sandbox for bug [https://github.com/reactjs/react-router/issues/3241](https://github.com/reactjs/react-router/issues/3241)



# Install



```
npm install
sudo npm install -g webpack
sudo npm install -g http-server
webpack
http-server --p 8080 --cors
http-server --p 9000 --cors
```


# Result

- [http://localhost:8080/](http://localhost:8080/)

Here the app starts without any problem and the "active-links" example runs successfully


- [http://localhost:9000/](http://localhost:9000/)

Here the app does not start at all

This is because on app mounting, it seems React-Router tries to create lazily the unexisting "storage key" if it does not already exists

```javascript
	  function getCurrentLocation(historyState) {
	    historyState = historyState || window.history.state || {};

	    var path = _DOMUtils.getWindowPath();
	    var _historyState = historyState;
	    var key = _historyState.key;

	    var state = undefined;
	    if (key) {
	      state = _DOMStateStorage.readState(key);
	    } else {
	      state = null;
	      key = history.createKey();

	      if (isSupported) window.history.replaceState(_extends({}, historyState, { key: key }), null, path);
	    }

	    var location = _PathUtils.parsePath(path);

	    return history.createLocation(_extends({}, location, { state: state }), undefined, key);
	  }
```


During this process, the base url of the page is not used and it makes the history.replaceState fails.


I've already encountered this problem in the past where I build myself my app router (still in production today) and the way to prevent this is to always use absolute urls in replaceState.

# Solving the problem

Replacing the incriminated solves the startup problem:

```javascript
      if (isSupported) window.history.replaceState(_extends({}, historyState, { key: key }), null, window.location.href);
```

But now, clicking on links do lead to the error again, but hopefully the basename works fine here:

To make it work for port 9000, you have to uncomment this basename line:

```javascript
const history = useRouterHistory(createHistory)({
  //basename: 'http://localhost:9000'
});
```

With these 2 things (fixing startup replaceState + using basename setting, I am able to run the app successfully)









