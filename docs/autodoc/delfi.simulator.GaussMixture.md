## **GaussMixture**`#!py3 class` { #GaussMixture data-toc-label=GaussMixture }


### *GaussMixture*.**\_\_init\_\_**`#!py3 (self, dim=1, noise_cov=[1.0, 0.1], bimodal=False, return_abs=False, seed=None)` { #\_\_init\_\_ data-toc-label=\_\_init\_\_ }


```
Gaussian Mixture simulator

Toy model that draws data from a mixture distribution with 2 components
that have mean theta and fixed noise.

Parameters
----------
dim : int
    Number of dimensions of parameters
noise_cov : list
    Covariance of noise on observations
seed : int or None
    If set, randomness is seeded
```


??? info "Source Code" 
	```py3 linenums="1 1 2" 
	def __init__(self, dim=1, noise_cov=[1.0, 0.1], bimodal=False, return_abs=False, seed=None):
	    
	    super().__init__(dim_param=dim, seed=seed)
	    self.a = [0.5, 0.5]  # mixture weights
	    self.noise_cov = [nc*np.eye(dim) for nc in noise_cov]
	    self.bimodal = bimodal
	    self.return_abs = return_abs
	
	```
### *GaussMixture*.**gen**`#!py3 (self, params_list, n_reps=1, pbar=None)` { #gen data-toc-label=gen }


```
Forward model for simulator for list of parameters

Parameters
----------
params_list : list of lists or 1-d np.arrays
    List of parameter vectors, each of which will be simulated
n_reps : int
    If greater than 1, generate multiple samples given param
pbar : tqdm.tqdm or None
    If None, will do nothing. Otherwise it will call pbar.update(1)
    after each sample.

Returns
-------
data_list : list of lists containing n_reps dicts with data
    Repetitions are runs with the same parameter set, different
    repetitions. Each dictionary must contain a key data that contains
    the results of the forward run. Additional entries can be present.
```


??? info "Source Code" 
	```py3 linenums="1 1 2" 
	def gen(self, params_list, n_reps=1, pbar=None):
	    
	    data_list = []
	    for param in params_list:
	        rep_list = []
	        for r in range(n_reps):
	            rep_list.append(self.gen_single(param))
	        data_list.append(rep_list)
	        if pbar is not None:
	            pbar.update(1)
	
	    return data_list
	
	```
### *GaussMixture*.**gen\_newseed**`#!py3 (self)` { #gen\_newseed data-toc-label=gen\_newseed }


```
Generates a new random seed
```


??? info "Source Code" 
	```py3 linenums="1 1 2" 
	def gen_newseed(self):
	    
	    if self.seed is None:
	        return None
	    else:
	        return self.rng.randint(0, 2**31)
	
	```
### *GaussMixture*.**gen\_single**`#!py3 (self, param)` { #gen\_single data-toc-label=gen\_single }


```
Forward model for simulator for single parameter set

Parameters
----------
params : list or np.array, 1d of length dim_param
    Parameter vector

Returns
-------
dict : dictionary with data
    The dictionary must contain a key data that contains the results of
    the forward run. Additional entries can be present.
```


??? info "Source Code" 
	```py3 linenums="1 1 2" 
	@copy_ancestor_docstring
	def gen_single(self, param):
	    # See BaseSimulator for docstring
	    param = np.asarray(param).reshape(-1)
	    assert param.ndim == 1
	    assert param.shape[0] == self.dim_param
	
	    if self.bimodal:
	        sample = dd.MoG(a=self.a, ms=[ (-1)**p * param for p in range(2)],
	                        Ss=self.noise_cov, seed=self.gen_newseed()).gen(1)
	    else:
	        sample = dd.MoG(a=self.a, ms=[param for p in range(2)],
	                        Ss=self.noise_cov, seed=self.gen_newseed()).gen(1)
	    if self.return_abs:
	        sample = np.abs(sample)
	
	    return {'data': sample.reshape(-1)}
	
	```
### *GaussMixture*.**reseed**`#!py3 (self, seed)` { #reseed data-toc-label=reseed }


```
Reseeds the distribution's RNG
```


??? info "Source Code" 
	```py3 linenums="1 1 2" 
	def reseed(self, seed):
	    
	    self.rng.seed(seed=seed)
	    self.seed = seed
	
	```
