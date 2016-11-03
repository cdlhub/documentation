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
 
These standards in methodology drove the design of the specification and so a different approach in any of these areas might have resulted in
substantially different data streams. However there is substantial scope to adapt ktools to support variations in model calculation methodology,
subject to adhering broadly to this calculation framework. 

#### 'Plug and play'

Oasis is an agnostic plug and play framework for loss models is achieved in ktools by separating the calculation steps into separate
components.  Each component is compiled separately which means any component can be replaced by a bespoke calculation component.  Further more,
the developer is free to decide on the internal data structures.
