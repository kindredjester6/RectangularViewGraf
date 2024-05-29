<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import * as makeNode from "../../lib/classNode";

    let tooltipText = "";
    let searchID = ""; // Variable to store the search ID

    onMount(() => {
        d3.csv('/distritos.csv').then(data => {
            console.log(data);

            data = data.map(item => {
                return {
                    id: item.ID,
                    size: item.POBL_2022
                };
            });

            let rootNode = makeNode.convertToHierarchy(data, "distritos");
            var root = d3.hierarchy(rootNode);
            const treeLayout = d3.cluster().size([360, 800]);
            treeLayout(root);

            const svg = d3.select('#RectangularViewDist')
                .call(d3.zoom().on("zoom", zoomed));

            const g = svg.append("g").attr("transform", "translate(850,850)");

            const nodesGroup = g.append("g").attr("class", "nodes");
            const linksGroup = g.append("g").attr("class", "links");

            // Initial draw
            update(root, 1);

            function update(source, depthLimit) {
                const descendants = source.descendants().filter(d => d.depth <= depthLimit);
                const links = source.links().filter(d => d.target.depth <= depthLimit);

                // Update nodes
                const nodes = nodesGroup.selectAll('circle.node')
                    .data(descendants, d => d.id);

                nodes.enter().append('circle')
                    .attr('class', 'node')
                    .attr('cx', 0)
                    .attr('cy', d => -d.y)
                    .attr('r', 5)
                    .attr("fill", "orange")
                    .attr('stroke', "darkgray")
                    .attr('stroke-width', 1)
                    .attr("transform", d => `rotate(${d.x}, 0, 0)`)
                    .on("mouseover", (event, d) => {
                        tooltipText = `ID: ${d.data.id}, Size: ${d.data.size}`;
                    })
                    .on("mouseout", () => {
                        tooltipText = "";
                    })
                    .merge(nodes)
                    .attr("fill", d => d.data.id === searchID ? "red" : "orange"); // Highlight the searched node

                nodes.exit().remove();

                // Update links
                const lineGen = d3.lineRadial()
                    .angle(d => d.x * Math.PI / 180)
                    .radius(d => d.y);

                const linkGen = d3.linkRadial()
                    .angle(d => d.x * Math.PI / 180)
                    .radius(d => d.y);

                const linksUpdate = linksGroup.selectAll('path.link')
                    .data(links, d => `${d.source.id}-${d.target.id}`);

                linksUpdate.enter().append("path")
                    .attr('class', 'link')
                    .attr('stroke', "darkgray")
                    .attr('stroke-width', 2)
                    .attr("d", linkGen)
                    .merge(linksUpdate)
                    .attr("d", d => lineGen([d.target, d.source]));

                linksUpdate.exit().remove();
            }

            function zoomed(event) {
                const transform = event.transform;
                const newDepthLimit = Math.max(1, Math.round(Math.log2(transform.k + 1) * 3)); // Adjusted zoom factor
                update(root, newDepthLimit);
                g.attr("transform", transform);
            }

            function searchNode() {
                update(root, Infinity); // Redraw the graph with the new search ID
            }

            // Expose the searchNode function to the Svelte context
            window.searchNode = searchNode;
        })
        .catch(error => {
            console.error('Error al cargar el archivo JSON:', error);
        });
    });
</script>

<input type="text" bind:value={searchID} placeholder="Enter node ID">
<button on:click={() => searchNode()}>Search</button>

<svg id="RectangularViewDist" width=1700 height=1700>
    <g transform="translate(850,850)">
        <g class="links"></g>
        <g class="nodes"></g>
    </g>
    <g id="tooltip">
        {#if tooltipText !== ""}
            <rect x="0" y="0" width="150" height="30" fill="white" stroke="black"></rect>
            <text x="5" y="20">{tooltipText}</text>
        {/if}
    </g>
</svg>
