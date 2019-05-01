<div style="display: flex">
<div>

<h1><u>Summary</u></h1>
<br/>

### Local zones

**Question**: How to handle time zone changes during a Python run?

- No support

- Allow datetime offsets to change

- Re-implement, avoiding system calls

</div>

<div style="flex-grow: 1"></div>
<img src="external-images/bharathi-kannan-sad-puppy.jpg"
     alt="An image of a sad puppy"
     class="fragment"
     height="350px"
     style="border: 2px solid"
     />

<br/>
<br/>
</div>

### IANA Zones

**Proposal**:

- Support `tzfile` and use heuristics to solve DST problem

- Data comes from the system, fallback to PyPI package ("`ensuretzdata`")

- Write a PEP to document the trade-offs

**Question**: How to ensure timely updates of PyPI package?
