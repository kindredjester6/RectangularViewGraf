
<script>
    import { onMount } from "svelte";
    import * as d3 from "d3";
    import * as makeNode from "../../lib/classNode"

    onMount(() => {
        d3.csv('/flareClass.csv').then(data => {
        console.log(data)

        data = data.map(item => {
            return {
            id: item.id,
            size: item.value
            };
        });

        let rootNode = makeNode.convertToHierarchy(data,"flare")
        var root =  d3.hierarchy(rootNode)


        const treeLayout = d3.cluster().size([360, 800]);
        treeLayout(root);
        const svg = d3.select('#RectangularViewFlare');

        console.log(root)
        // Dibuja los enlaces (líneas)
        svg.select('g.nodes')
        .selectAll('circle.node')
        .data(root.descendants())
        .enter()
        .append('circle')
        .classed('node', true)
        .attr('cx', 0)
        .attr('cy', d => (-d.y))
        .attr('r', 5)
        .attr("fill", "orange")
        .attr('stroke', "darkgray")
        .attr('stroke-width', 1)
        .attr("transform", d => `
            rotate(${d.x/2}, 0, 0)
        `);
        //.text((d) => d.id)
        //.style("font-size", "2000px");

        // Dibuja los nodos (círculos)
        var lineGen = d3.lineRadial()
        .angle(d => (d.x * Math.PI / 180)/2)
        .radius(d => d.y);

        var linkGen = d3.linkRadial()
        .angle(d => d.x * Math.PI / 180)
        .radius(d => d.y);

        console.log(root.links().target)
        // draw links
        svg.select('g.links')
        .selectAll('path.link')
        .data(root.links())
        .enter()
        .append("path")
        .classed('link', true)
        .attr('stroke', "darkgray")
        .attr('stroke-width', 2)
        .attr("d", linkGen)
        .attr("d", (d) => lineGen([d.target, d.source]))

        svg.selectAll('.label')
        .data(root.descendants())
        .enter()
        .append('text')
        .attr('class', 'label')
        .attr('x', d => d.y )
        .attr('y', d => d.x)
        .text(d => d.id);
        })
        .catch(error => {
            console.error('Error al cargar el archivo JSON:', error);
        });
    })
</script>

<svg id="RectangularViewFlare" width=1700 height=1700>
    <g transform="translate(850,850)">
      <g class="rects"></g>
      <g class="links"></g>
      <g class="nodes"></g>
    </g>
</svg>