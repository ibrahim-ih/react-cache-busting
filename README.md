# react-cache-busting

## Usage

### Class Component
```javascript
import React from 'react'
import PropTypes from 'prop-types'
import packageJson from '../package.json'

global.appVersion = packageJson.version

// version from response - first param, local version second param
const semverGreaterThan = (versionA, versionB) => {
    const versionsA = versionA.split(/\./g)

    const versionsB = versionB.split(/\./g)
    while (versionsA.length || versionsB.length) {
        const a = Number(versionsA.shift())

        const b = Number(versionsB.shift())
        // eslint-disable-next-line no-continue
        if (a === b) continue
        // eslint-disable-next-line no-restricted-globals
        return a > b || isNaN(b)
    }
    return false
}

class CacheBusting extends React.Component {
    static propTypes = {
        children: PropTypes.oneOfType([
            PropTypes.array,
            PropTypes.object,
            PropTypes.node,
            PropTypes.func,
        ]),
    }

    static defaultProps = {
        children: null,
    }

    constructor(props) {
        super(props)
        this.state = {
            loading: true,
            isLatestVersion: false,
            refreshCacheAndReload: () => {
                console.log('Clearing cache and hard reloading...')
                if (caches) {
                    // Service worker cache should be cleared with caches.delete()
                    // eslint-disable-next-line func-names
                    caches.keys().then(function(names) {
                        for (const name of names) caches.delete(name)
                    })
                }

                // delete browser cache and hard reload
                window.location.reload(true)
            },
        }
    }

    componentDidMount() {
        fetch('/meta.json')
            .then(response => response.json())
            .then(meta => {
                const latestVersion = meta.version
                const currentVersion = global.appVersion

                const shouldForceRefresh = semverGreaterThan(
                    latestVersion,
                    currentVersion
                )
                if (shouldForceRefresh) {
                    console.log(
                        `We have a new version - ${latestVersion}. Should force refresh`
                    )
                    this.setState({ loading: false, isLatestVersion: false })
                } else {
                    console.log(
                        `You already have the latest version - ${latestVersion}. No cache refresh needed.`
                    )
                    this.setState({ loading: false, isLatestVersion: true })
                }
            })
    }

    render() {
        const { loading, isLatestVersion, refreshCacheAndReload } = this.state
        const { children } = this.props
        return children({
            loading,
            isLatestVersion,
            refreshCacheAndReload,
        })
    }
}

export default CacheBuster

```

### Functional Component

```javascript
import React, { useState, useEffect } from 'react';
import PropTypes from 'prop-types';
import packageJson from '../package.json';

global.appVersion = packageJson.version;

const semverGreaterThan = (versionA, versionB) => {
  const versionsA = versionA.split(/\./g);
  const versionsB = versionB.split(/\./g);
  while (versionsA.length || versionsB.length) {
    const a = Number(versionsA.shift());
    const b = Number(versionsB.shift());
    if (a === b) continue;
    return a > b || isNaN(b);
  }
  return false;
};

function CacheBusting({ children }) {
  const [loading, setLoading] = useState(true);
  const [isLatestVersion, setIsLatestVersion] = useState(false);

  const refreshCacheAndReload = () => {
    console.log('Clearing cache and hard reloading...');
    if (caches) {
      caches.keys().then(names => {
        for (const name of names) caches.delete(name);
      });
    }
    window.location.reload(true);
  };

  useEffect(() => {
    fetch('/meta.json')
      .then(response => response.json())
      .then(meta => {
        const latestVersion = meta.version;
        const currentVersion = global.appVersion;
        const shouldForceRefresh = semverGreaterThan(
          latestVersion,
          currentVersion
        );
        if (shouldForceRefresh) {
          console.log(`We have a new version - ${latestVersion}. Should force refresh`);
          setIsLatestVersion(false);
        } else {
          console.log(`You already have the latest version - ${latestVersion}. No cache refresh needed.`);
          setIsLatestVersion(true);
        }
        setLoading(false);
      });
  }, []);

  return children({
    loading,
    isLatestVersion,
    refreshCacheAndReload,
  });
}

CacheBusting.propTypes = {
  children: PropTypes.func.isRequired,
};

export default CacheBusting;

```
