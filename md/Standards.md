# Standards

The element of ktools that represents a standard is the Specification, not the input data structures in the reference model, 
nor even the specific logic that drives the calculations. The standard is essentially the structure of data streams that flow in between
the calculation components. 

### Loss modelling methodology

The data stream structures arise from the Oasis methodology for representing models and calculating losses which can be summarized as follows;

* **Event-based calculation**. An event is a data representation of a catastophic event. Oasis calculates losses to a set of exposures on an event 
by event basis, and so the calculation is driven by a list of events. Work is also distributed by events. The smallest unit of work is a loss calculation
for a single event.
* **Discrete probability distributions**. Probability distributions for intensity and damage are represented as histograms with bins which represent 
either point values or ranges of intensity or damage. If the source model format uses closed form distributions then these need to be converted 
to a discrete distribution and an appropriate discretization defined.
* **Effective damageability**. Oasis performs a discrete convolution of the intensity distribution with the damage distribution to create a single
effective damageability distribution, held as a discrete cumulative distribution in the data stream. This distribution combines both sources 
of uncertainty in a single distribution, representing the overall damage distribution conditional on the event occurring.
* **Monte-Carlo sampling**. Oasis performs Monte-Carlo sampling of ground up and insured losses as its loss calculation methodology.
 
These standards in methodology drove the design of the specification and so a different approach in any of these areas would have resulted in
substantially different data streams. However there is a lot of scope to adapt ktools to support variations in model calculation methodology,
subject to them adhering broadly to this calculation framework. 

### The 'plug and play' kernel

Oasis is an agnostic plug and play framework for loss models, which is achieved in ktools by separating the calculation steps into separate components.  Each component is compiled separately which means any component can be replaced by a bespoke calculation component.  Furthermore, the component developer has a free hand to decide on the input data structures.  The components that are most likely to be adapted for model variations are;

* **getmodel**. There is great variation in the way uncertainty is expressed in catastrophe models, and more efficient ways of storing data. The reference implementation of getmodel reads in hazard footprint and damage probability distribution data which has already been discretized and converted Oasis 'kernel' format.   getmodel can be adapted to read in the model data in the models source format, and perform the discretization in the calculation logic.

* **gulcalc**. The reference implementation has a simple approach to sampling damage using simple uniform random numbers, and to correlating damage across items.  gulcalc could be adapted to more sophisticated sampling strategies and methods of correlating damage.  Also, in respect of multi-peril damage, different methodologies of combining damage from different perils to the same exposure such that the total insured value or replacement value is not exceeded could be implemented.  This is also the place where conditional coverage damage methodologies could be implemented. 

From a users perspective, additional calculation components could be developed to generate a particular type of report, and fmcalc could be adapted to support different policy types by adding new calculation rules and defining different data structures for policy data.
