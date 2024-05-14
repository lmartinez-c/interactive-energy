<script>
    import { onMount, afterUpdate } from 'svelte';
    import * as d3 from 'd3';

    export let yearRangeMin;
    export let yearRangeMax;
    export let populationMin;
    export let selectedCategory;
    export let dataType;
    export let updateCategoryLabel;
    let categoryLabel;
    let data = [];

    const countries = new Set([
        "Afghanistan", "Albania", "Algeria", "Andorra", "Angola", "Antigua and Barbuda",
        "Argentina", "Armenia", "Australia", "Austria", "Azerbaijan", "Bahamas",
        "Bahrain", "Bangladesh", "Barbados", "Belarus", "Belgium", "Belize",
        "Benin", "Bhutan", "Bolivia", "Bosnia and Herzegovina", "Botswana", "Brazil",
        "Brunei", "Bulgaria", "Burkina Faso", "Burundi", "Cabo Verde", "Cambodia",
        "Cameroon", "Canada", "Central African Republic", "Chad", "Chile", "China",
        "Colombia", "Comoros", "Congo", "Costa Rica", "Croatia", "Cuba", "Cyprus",
        "Czech Republic", "Denmark", "Djibouti", "Dominica", "Dominican Republic",
        "Ecuador", "Egypt", "El Salvador", "Equatorial Guinea", "Eritrea", "Estonia",
        "Eswatini", "Ethiopia", "Fiji", "Finland", "France", "Gabon", "Gambia",
        "Georgia", "Germany", "Ghana", "Greece", "Grenada", "Guatemala", "Guinea",
        "Guinea-Bissau", "Guyana", "Haiti", "Honduras", "Hungary", "Iceland", "India",
        "Indonesia", "Iran", "Iraq", "Ireland", "Israel", "Italy", "Jamaica", "Japan",
        "Jordan", "Kazakhstan", "Kenya", "Kiribati", "Kuwait", "Kyrgyzstan", "Laos",
        "Latvia", "Lebanon", "Lesotho", "Liberia", "Libya", "Liechtenstein", "Lithuania",
        "Luxembourg", "Madagascar", "Malawi", "Malaysia", "Maldives", "Mali", "Malta",
        "Marshall Islands", "Mauritania", "Mauritius", "Mexico", "Micronesia", "Moldova",
        "Monaco", "Mongolia", "Montenegro", "Morocco", "Mozambique", "Myanmar",
        "Namibia", "Nauru", "Nepal", "Netherlands", "New Zealand", "Nicaragua", "Niger",
        "Nigeria", "North Macedonia", "Norway", "Oman", "Pakistan", "Palau", "Panama",
        "Papua New Guinea", "Paraguay", "Peru", "Philippines", "Poland", "Portugal",
        "Qatar", "Romania", "Russia", "Rwanda", "Saint Kitts and Nevis", "Saint Lucia",
        "Saint Vincent and the Grenadines", "Samoa", "San Marino", "Sao Tome and Principe",
        "Saudi Arabia", "Senegal", "Serbia", "Seychelles", "Sierra Leone", "Singapore",
        "Slovakia", "Slovenia", "Solomon Islands", "Somalia", "South Africa", "South Sudan",
        "Spain", "Sri Lanka", "Sudan", "Suriname", "Sweden", "Switzerland", "Syria", "Taiwan",
        "Tajikistan", "Tanzania", "Thailand", "Timor-Leste", "Togo", "Tonga", "Trinidad and Tobago",
        "Tunisia", "Turkey", "Turkmenistan", "Tuvalu", "Uganda", "Ukraine", "United Arab Emirates",
        "United Kingdom", "United States", "Uruguay", "Uzbekistan", "Vanuatu", "Vatican City", "Venezuela",
        "Vietnam", "Yemen", "Zambia", "Zimbabwe"
    ]);

    onMount(async () => {
        data = await d3.csv('/energy.csv');
        data.forEach(d => {
            d.year = +d.year;
            d.population = +d.population;
            Object.keys(d).forEach(key => {
                if (key.endsWith('_consumption') || key.endsWith('_production')) {
                    d[key] = +d[key];
                }
            });
        });
        visualizeData();
    });

    afterUpdate(() => {
        visualizeData();
    });

    function visualizeData() {
        categoryLabel = `Top Countries by ${selectedCategory.replace('_', ' ')} ${dataType.charAt(0).toUpperCase() + dataType.slice(1)}`.toUpperCase();
        const svg = d3.select('#visualization svg');
        const width = 1200;
        const height = 800;
        const margin = { top: 40, right: 40, bottom: 60, left: 60 };
        const innerWidth = width - margin.left - margin.right;
        const innerHeight = height - margin.top - margin.bottom;
        const centerX = innerWidth / 2;
        const centerY = innerHeight / 2;
        const boundaryRadius = Math.min(innerWidth, innerHeight) / 2 - 50;

        const colorScale = d3.scaleLinear()
            .domain([0, d3.max(data, d => d[`${selectedCategory}_${dataType}`])])
            .range(["#8000ff", "#7fff00"]); 

        const radiusScale = d3.scaleSqrt().range([10, boundaryRadius / 3]);

        const categoryKey = `${selectedCategory}_${dataType}`;
        
        const aggregatedData = d3.rollups(
            data.filter(d => {
                return countries.has(d.country) &&
                    d.year >= yearRangeMin && d.year <= yearRangeMax &&
                    d.population >= populationMin;
            }),
            v => d3.sum(v, d => d[categoryKey]),
            d => d.country
        ).map(([country, value]) => ({ country, value }));

        const top20Data = aggregatedData.sort((a, b) => b.value - a.value).slice(0, 20);

        radiusScale.domain([0, d3.max(top20Data, d => d.value)]);

        svg.selectAll('*').remove();

        const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`);

        const tooltip = d3.select('body').append('div')
            .attr('class', 'tooltip')
            .style('opacity', 0)
            .style('position', 'absolute')
            .style('pointer-events', 'none')
            .style('background', 'rgba(0, 0, 0, 0.8)')
            .style('border-radius', '8px')
            .style('padding', '8px')
            .style('color', '#d9d9d9') 
            .style('font-family', 'monospace');

        const circles = g.selectAll('circle.data-circle')
            .data(top20Data)
            .enter().append('circle')
            .attr('class', 'data-circle')
            .attr('r', d => radiusScale(d.value))
            .attr('fill', d => colorScale(d.value))
            .attr('stroke', '#000') 
            .attr('stroke-width', 1.5)
            .on('mouseover', (event, d) => {
                tooltip.transition().duration(200).style('opacity', .9);
                tooltip.html(`Country: ${d.country}<br>Value: ${d.value}`)
                    .style('left', (event.pageX + 5) + 'px')
                    .style('top', (event.pageY - 28) + 'px');
            })
            .on('mousemove', (event) => {
                tooltip
                    .style('left', (event.pageX + 5) + 'px')
                    .style('top', (event.pageY - 28) + 'px');
            })
            .on('mouseout', () => {
                tooltip.transition().duration(500).style('opacity', 0);
            });

        const labels = g.selectAll('.label')
            .data(top20Data)
            .enter().append('text')
            .attr('class', 'label')
            .attr('dy', '0.35em')
            .attr('font-size', '10px')
            .attr('fill', '#000') 
            .attr('text-anchor', 'middle')
            .text(d => d.country)
            .on('mouseover', (event, d) => {
                tooltip.transition().duration(200).style('opacity', .9);
                tooltip.html(`Country: ${d.country}<br>Value: ${d.value}`)
                    .style('left', (event.pageX + 5) + 'px')
                    .style('top', (event.pageY - 28) + 'px');
            })
            .on('mousemove', (event) => {
                tooltip
                    .style('left', (event.pageX + 5) + 'px')
                    .style('top', (event.pageY - 28) + 'px');
            })
            .on('mouseout', () => {
                tooltip.transition().duration(500).style('opacity', 0);
            });

        const simulation = d3.forceSimulation(top20Data)
            .force('charge', d3.forceManyBody().strength(1))
            .force('center', d3.forceCenter(centerX, centerY))
            .force('collision', d3.forceCollide().radius(d => radiusScale(d.value) + 1).strength(1))
            .force('x', d3.forceX(centerX).strength(0.05))
            .force('y', d3.forceY(centerY).strength(0.05))
            .on('tick', () => {
                circles
                    .attr('cx', d => d.x)
                    .attr('cy', d => d.y);
                labels
                    .attr('x', d => d.x)
                    .attr('y', d => d.y);
            });

        g.append('text')
            .attr('class', 'axis-label')
            .attr('transform', `translate(${innerWidth / 2}, ${-20})`)
            .style('text-anchor', 'middle')
            .style('font-size', '24px')
            .style('font-family', 'monospace')
            .style('padding', '10px')
            .style('border', '2px solid #00e5ff')
            .style('background', 'rgba(0, 0, 0, 0.8)')
            .style('border-radius', '8px')
            .style('color', '#00e5ff')
            .text(categoryLabel);

        
        g.append('text')
            .attr('class', 'description-label')
            .attr('transform', `translate(${innerWidth / 2}, ${0})`)
            .style('text-anchor', 'middle')
            .style('font-size', '16px')
            .style('font-family', 'monospace')
            .style('color', '#000')
            .text("Measured in terawatt-hours.");
    }
</script>

<style>
    html, body {
        background-color: #f0f0f0;
        color: #00e5ff;
        font-family: 'Roboto', sans-serif;
        height: 100%;
        margin: 0;
        padding: 0;
    }

    .tooltip {
        text-align: center;
        width: auto;
        height: auto;
        font: 12px 'Roboto', sans-serif;
        pointer-events: none;
        opacity: 0;
        transition: opacity 0.2s;
    }

    select {
        border: 1px solid #d9d9d9; 
        background-color: #444;
        color: #d9d9d9; 
        padding: 5px;
        font-family: 'Roboto', sans-serif;
    }

    label {
        color: #000; 
    }

    .slider-label {
        color: #000; 
    }

    input[type="range"] {
        accent-color: #808080; 
    }
</style>

<svg width="1200" height="800"></svg>
