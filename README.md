# my-library
Creating a library (React,typescript, webpack,babel) and linking it with other React app to share utils and components


# Create library folder struct similar to:
my-shared-lib/
├── src/
│   ├── components/
│   │   └── Button.tsx
│   ├── utils/
│   │   └── formatDate.ts
│   └── index.ts
├── dist/
├── tsconfig.json
├── package.json
├── webpack.config.js
├── .babelrc
├── .gitignore
└── README.md

# package.json
- "npm init -y" to create package.json file
- install all the dependencies and dev-dependencies
- add script `"build": "webpack --mode production"`
- Avoid using react and react-dom as internal dependencies in the library — mark them as peerDependencies in your package.json to avoid duplicate React issues. Tells the consumer to install them.
- even after linking got error "Module not found: Error: Can't resolve 'my-shared-lib' in <>" in application using the library ->  modified "main": "index.js" to "main": "dist/index.js" to point to correct entry point.

# webpack.config.js
- path: Node.js built-in module to handle file paths.
- webpack-node-externals: Prevents bundling node_modules dependencies — instead, marks them as external (so the host app provides them). This avoids duplicating react, react-dom, etc.
- entry - Entry point of your library.This is where Webpack starts building the dependency graph.
- output
    - path: Output folder (dist).
    - filename: Name of the output bundle.
    - library: Configures the bundle to be a library:
        - name: Global variable name when loaded via <script> tag.
        - type: 'umd': Universal Module Definition — makes your library work with:
            -CommonJS (Node)
            -AMD (RequireJS)
            -Global variable (Browser)
    - clean: true: Clears the dist/ folder before building.
- resolve : Allows imports without needing to write file extensions.
- module 
    - rules
        - Uses babel-loader to transpile TypeScript + React JSX using Babel.
        - Allows importing CSS in components

NOTE: 
If you bundle React into your library, and the host app already has it, you’ll likely get errors like:
    - Invalid hook call
    - React is not defined
    - Conflicting react versions
Using externals and peerDependencies avoids this problem by sharing React across your MFEs and shared library.
You can also manually externalize only specific modules:

If fine grain neede :
externals: {
  react: 'React',
  'react-dom': 'ReactDOM',
}
But nodeExternals() is better when you want to skip all node_modules

# to link the local library to an application
in lib folder
    - npm link
in app folder 
    - npm link my-shared-lib
    - check in node-modules folder and by "npm list" cmd
when there is a change in lib run "npm run build" and the new lib changes will be reflected in the app

# usage in other app (code)
import React, { useContext } from 'react';
import { ThemeContext } from '../context';
import { formatDate, Button } from 'nxo-library';


const Dashboard = () => {

    
    const context = useContext(ThemeContext);
    let newDate = new Date()


  if (!context) {
    throw new Error("HomePage must be used within a ThemeProvider");
  }

  const { theme, toggleTheme } = context;

  const style: React.CSSProperties = {
    backgroundColor: theme === "light" ? "#fff" : "#333",
    color: theme === "light" ? "#000" : "#fff",
    padding: "2rem",
  };

  return (
    <div style={style}>
      <h1>Current Theme: {theme}</h1>
      {/* <button onClick={toggleTheme}>Toggle Theme </button> */}
      <p>{formatDate(newDate)}</p>
      <Button label="Testing" onClick={toggleTheme}></Button>
    </div>
  );
};

export default Dashboard;