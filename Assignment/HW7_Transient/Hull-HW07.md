### Transients
  * Quinn Hull
  * 03/11/21
  * HW-07


### The Changes:
  1. Change `sp1` from 40 to 90
  2. Change `sim_years` from 20 to 100
  3. I'm a little confused about what to do from nper etc...
    * Return to 3.2 Temporal Discretization
    * RE: nper - the initial values are overwritten within 3.2
  4. Changed `H_init[:, :, 0]` to `H_init[:, :, -1] = 30 ` for right boundary
  5. Changed `K_horiz = 0.1 ` and `K_vert = 0.1`
  6. Changed `Q1 = 500`, and `Q2 = 0`
    * REturn to 8.1. Should first period be pumping or not?
  7. Changed `recharge = 5e-4 #m/day`, also changed so that it occurs over the entire domain
  8. Inconsistencies - chose the description in 'model description' - so 100 years, not 50 years. And recharge of 5e^-4
  9. Uncommented ET
  10. `5b.1` K_vals[:,:,:] = 0.1  (not = 1)
  11. Run 1:
    - It's really difficult to see the oscillations in head in Figure 1. I think it's because the pumping period is too long!
    - The gradient appears to be oriented in the wrong direction
    - Well is accidentally injecting (not pumping) (whoops)
      * Change - pump well to negative
  12. Run 2:
    This pumping rate really seems to cause issues.
    - Backed pumping off to 10 m^3 / day
    - Changed pumping period so that the first (non pumping) period is 270 days
  13. Run 3:
    - This really worked, repeated with -50
  14. Run 4:
    - This really worked, repeated with -500
  15. Run 5:
    - This doesn't really work. Seems to 'break' the model by having a drawdown that is very very deep!

### The Figures:
a) left panel showing the head at the well and right panel showing the head at the midpint of the domain, both as functions of time over the entire simulation.

*a.1 Head at well and moving average*
<img src="assets/Hull-HW07-bec0928d.png" width="400" />

*a.2 Head at well and moving average over two years*
<img src="assets/Hull-HW07-fff5456f.png" width="400" />

*a.3 Head at midpoint and moving average*
<img src="assets/Hull-HW07-029b2fc0.png" width="400" />

b)  The head along a transect between the constant head boundaries through the well at three times: the initial steady state; the final pump-on period; and the final pump-off period.

*b.1 Head along transect*
<img src="assets/Hull-HW07-d6166f5b.png" width="400" />

c) A contour map with flow vectors at three times: the initial steady state; the final pump-on period; and the final pump-off period.
*c.1 Contour Map 1, Head*
<img src="assets/Hull-HW07-d480eb5f.png" width="400" />

*c.2 Contour Map 2, Head*
<img src="assets/Hull-HW07-1aee529e.png" width="400" />

*c.3 Contour Map 3, Head*
<img src="assets/Hull-HW07-a802b558.png" width="400" />

d) A contour map of the drawdown calculated for two periods: between the initial steady state and the final  simulation time and between the final pump-on period and the final pump-off period.

*d.1 Contour Map 1, Drawdown*
<img src="assets/Hull-HW07-34f02927.png" width="400" />

*d.2 Contour Map 2, Drawdown*
<img src="assets/Hull-HW07-f6a65bae.png" width="400" />

### The Challenges
a) The gradient is not uniform for the initial steady state conditions - discuss the influences of recharge and the unconfined condition on this nonlinearity.
  - The gradient is not uniform primarily due to it being an unconfined aquifer. We have seen from previous exercises that this nonlinearity is a function of decreasing aquifer thickness (approximated in modflow like a decrease in K).
  - Changes in recharge rate will affect the decrease of convexity of the head curve, but not the shape of the curve itself <img src="assets/Hull-HW07-9be49ce6.png" width="200" />.


b) Determine if the system has reached steady state - consider a point at the well and another at the center of the domain.  
  - I think this is a question of not just where, but when. We can consider 'steady' state to be some point in time where over the course of a year the heads are no longer decreasing and have stabalized. From visual inspection, we know that must be some point after year 40 for both midpoint and pumping well, but they don't appear to be identical.
  - Steady state is also a question of 'how' (i.e., 'how steady'). The code snippets below extract a moving average from the midpoint and well, take the difference between the moving average at times t and t-1, and (using an arbitrary 'tolerance' where the difference between time-steps is approximately zero), pulls out the year at which we reach steady state.  According to this method:
    * `the first steady state year at well is, 67.0`
    * `the first steady state year at midpoint is, 66.0`
          # extract and plot head time series at specific points
          time_series1 = headarray[:,domain_centerpt[1],domain_centerpt[2]]
          time_series2 = headarray[:,well_loc[1],well_loc[2]]

          n_in = 8
          time_series1_mv = moving_average(time_series1,n_in)
          time_series2_mv = moving_average(time_series2,n_in)
          elyears_mv = elyears[int(n_in/2)-1:-int(n_in/2)]

          tol = 5e-5 # the tolerance to set as 'steady'
          # locate steady state
          time_series1_mv_diff = time_series1_mv[:-1] - time_series1_mv[1:]
          time_series2_mv_diff = time_series2_mv[:-1]  - time_series2_mv[1:]

          # first year where head does not change
          print('the first steady state year at well is,', elyears_mv[:-1][time_series1_mv_diff < tol][0])
          print('the first steady state year at midpoint is,', elyears_mv[:-1][time_series2_mv_diff <= tol][0])


c) Find the zone of influence of the well defined in two ways:
    - Based on the drawdown from the initial steady state to the end of simulation time (end of final no-pumping stress period)
      > The figure below shows that at the end of the last no-pump period, the drawdown is limited to the area immediately around the well

      <img src="assets/Hull-HW07-7894344b.png" width="200" />


    - Based on the drawdown from the end of the last pump-on stress period to the end of simulation time.
        > Figure d.1 shows that the entire domain is basically within the zone of influence after the final no-pumping stress period

d) How long does it take a point at the center of the domain to reach steady state.  At that point, explain how you could divide the domain into a steady and transient part and solve each separately.
    - It took ~ 60 years for a point in the middle of the domain (near pumping well) to reach steady state (see question d).
    - We would probably need to approach the entire model as a transient model for some period of time, but we could use figures like this one to show that further from the pumping well the model seems to approach steady more quickly than further from the well. (Unless I'm looking at that all wrong). I'd love to talk more about this question! <img src="assets/Hull-HW07-ad70ea1c.png" width="200" />


e) Find a constant pumping rate (same throughout the year) that matches the head time series at the middle of the domain.  

f) Find a constant pumping rate (same throughout the year) that matches the head time series at the well, leaving only a regular, repeating seasonal residual.  Are the two pumping rates the same?

g) Discuss the sources of water captured by this well.  If you're up for a challenge, calculate them for the final pump-on period!

h) Discuss how you would define the capture zone of the well.  How is it different than our definitions of capture zone so far in the course?

    - I'm not sure how to do either e-h. :(
