<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import * as makeNode from "../../lib/classNode";

    let tooltipText = "";

    onMount(() => {
        d3.csv('/flareClass.csv').then(data => {
            console.log(data);

            data = data.map(item => {
                return {
                    id: item.id,
                    size: item.value
                };
            });

            let rootNode = makeNode.convertToHierarchy(data, "flare");
            var root = d3.hierarchy(rootNode);
            const treeLayout = d3.cluster().size([360, 800]);
            treeLayout(root);

            const svg = d3.select('#RectangularViewFlare')
                .call(d3.zoom().on("zoom", zoomed));

            const g = svg.select("g");

            const nodesGroup = g.select(".nodes");
            const linksGroup = g.select(".links");

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
                    .attr('cy', d => (-d.y))
                    .attr('r', 5)
                    .attr("fill", "orange")
                    .attr('stroke', "darkgray")
                    .attr('stroke-width', 1)
                    .attr("transform", d => `rotate(${d.x / 2}, 0, 0)`)
                    .on("mouseover", (event, d) => {
                        tooltipText = `ID: ${d.data.id}, Size: ${d.data.size}`;
                    })
                    .on("mouseout", () => {
                        tooltipText = "";
                    })
                    .merge(nodes);

                nodes.exit().remove();

                // Update links
                const lineGen = d3.lineRadial()
                    .angle(d => (d.x * Math.PI / 180) / 2)
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
        })
        .catch(error => {
            console.error('Error al cargar el archivo JSON:', error);
        });
    });
</script>

<svg id="RectangularViewFlare" width=1700 height=1700>
    <g transform="translate(850,850)">
      <g class="rects"></g>
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
