# mgardos01.github.io
My personal blog, hosted at [mgardos01.github.io](https://mgardos01.github.io/), themed off of the Comittee of Zero website ([committeeofzero.github.io](https://github.com/CommitteeOfZero/committeeofzero.github.io)).

Built with [Zola](https://www.getzola.org/).

# Setup
- Install the following dependencies: 
  - [Node](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm)
  - [Zola](https://www.getzola.org/documentation/getting-started/installation/)
- Clone this repository
  ```sh
  $ git clone https://github.com/mgardos01/mgardos01.github.io.git
  ```
- Install npm packages (Just [skeleton-sass](https://www.npmjs.com/package/skeleton-sass-official))
  ```sh
  $ npm install
  ```
- Adjust variables in ```config.toml``` as needed. 
- Build the site in the ```public``` directory with 
  ```sh
  $ zola build
  ``` 
  To build the site **and** serve the site using a local server, run   
  ```sh 
  $ zola serve
  ```
  Refer to the Zola CLI [documentation](https://www.getzola.org/documentation/getting-started/cli-usage/) for more information.  