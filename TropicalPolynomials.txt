from math import inf

##################################################################
# Example usage:
# x = Monomial(0,{'x':1})
# x2 = Monomial(0,{'x':2})
# xy = Monomial(0,{'x':1, 'y':1})
# y = Monomial(0,{'y':1})
# y2 = Monomial(0,{'y':2})
# o = Monomial(0,{})
# x * xy
##################################################################

class Monomial:
    
    def __init__(self, coefficient, degree, debug=False):
        # coefficient should be a number or a polynomial/monomial instance
        # degree should be a dictionary of variables and their exponents
        
        self.coefficient = self.c = coefficient
        self.degree = self.d = degree
        self.debug = debug
        
    def __repr__(self):
        if self.degree == {}:
            name = str(self.coefficient)
        elif self.coefficient == 0:
            name = ''
        else:   
            name = str(self.coefficient)
        for variable in self.degree:
            if self.degree[variable] == 0:
                pass
            elif self.degree[variable] == 1:
                name = name + str(variable)
            else:
                name = name + str(variable) + "^" + str(self.degree[variable])
        return name
    
    def __add__(self, other):
        if self.debug:
            print("adding " + str(self) + " to " + str(other))
        if isinstance(other, Monomial):
            if self.debug:
                print("is monomial") 
            if self.degree == other.degree:
                return Monomial(max(self.c, other.c), self.degree)
            else:
                return Polynomial([self, other])
        if isinstance(other, int):
            if math.isinf(other):
                return self
            return self + Monomial(other,{})
        else:
            if self.debug:
                print("trying other add")
            return other + self
    
    # def __call__(self, *args):
        # should be partial function application, maybe pass in a dictionary?
        
    def __mul__(self, other):
        if self.debug:
            print("multiplying " + str(self) + " to " + str(other))
        if isinstance(other, int):
            return Monomial(self.coefficient + other, self.degree)  
        if isinstance(other, Monomial):
            degree = {}
            for mon in other, self:
                for key in mon.degree:
                    if key in degree:
                        degree[key] += mon.degree[key]
                    else:
                        degree[key] = mon.degree[key]
            return Monomial(self.coefficient + other.coefficient, degree)
        if isinstance(other, Polynomial):
            return other * self
        
    def __rmul__(self, other):
        return self * other
    
    def inv(self):
        return Monomial(-self.coefficient, {key:-self.degree[key] for key in self.degree})

class Polynomial:
    def __init__(self, monomials, debug=False):
        self.monomials = monomials
        self.debug = debug
        
    def __repr__(self):
        string = ""
        for monomial in self.monomials:
            string += monomial.__repr__()
            string += " + "
        return string[:-3] # removes extra characters
    
    def __call__(self, *args):
        return sum([m(args) for m in monomials])
    
    def __add__(self, other):
        if self.debug:
            print("adding " + str(self) + " to " + str(other))
        if isinstance(other, int):
            return self + Monomial(other, (0,0))
        if isinstance(other, Monomial):
            added = False
            monomials = []
            for m in self.monomials:
                if m.degree == other.degree:
                    monomials.append(Monomial(max(other.coefficient, m.coefficient), m.degree))
                    added = True
                else:
                    monomials.append(m)
            if not added:
                monomials.append(other)
            return Polynomial(monomials)
        if isinstance(other, Polynomial):
            result = Polynomial(self.monomials)
            for monomial in other.monomials:
                result = result + monomial
            return result
        else:
            print("Polynomial doesn't know how to add that")
    
    def __mul__(self, other):
        if isinstance(other, int):
            return Polynomial([monomial * other for monomial in self.monomials])
        if isinstance(other, Monomial):
            if self.debug:
                print('multiplying monomial with polynomial')
                print('multiplying ' + str(self) + ' to ' + str(other))
            monomials = [monomial * other for monomial in self.monomials]
            return Polynomial(monomials)
        if isinstance(other, Polynomial):
            if self.debug:
                print('multiplying polynomial with polynomial')
                print('multiplying ' + str(self) + ' to ' + str(other))
            result = other * self.monomials[0]
            for monomial in self.monomials[0:]:
                result += other * monomial
            return result
        
    def remove_infinite_terms(self):
        cleaned = []
        for monomial in self.monomials:
            if not math.isinf(monomial.coefficient):
                cleaned.append(monomial)
        self.monomials = cleaned


# In[116]:



