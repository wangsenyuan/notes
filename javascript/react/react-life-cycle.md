1. static getDerivedStateFromProps(nextProps, prevState) 
   It’s called before the render() method during the mounting phase, and before the update phase. It returns either an object to update the state of a component, or null when there’s nothing to update.
2. 