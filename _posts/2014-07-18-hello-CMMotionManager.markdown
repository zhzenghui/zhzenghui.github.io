---
layout: post
title:  "关于iOS 设备方向角度问题!"
date:   2014-8-8 19:7:27
categories: CMMotionManager update
---

CMMotionManager
Look at gravity:

self.deviceQueue = [[NSOperationQueue alloc] init];
self.motionManager = [[CMMotionManager alloc] init];
self.motionManager.deviceMotionUpdateInterval = 5.0 / 60.0;

// UIDevice *device = [UIDevice currentDevice];

[self.motionManager startDeviceMotionUpdatesUsingReferenceFrame:CMAttitudeReferenceFrameXArbitraryZVertical
                                                        toQueue:self.deviceQueue
                                                    withHandler:^(CMDeviceMotion *motion, NSError *error)
{
    [[NSOperationQueue mainQueue] addOperationWithBlock:^{
        CGFloat x = motion.gravity.x;
        CGFloat y = motion.gravity.y;
        CGFloat z = motion.gravity.z;
    }];
}];
With this reference frame (CMAttitudeReferenceFrameXArbitraryZVertical), if z is near zero, you're holding it on a plane perpendicular with the ground (e.g. as if you were holding it against a wall) and as you rotate it on that plane, x and y values change. Vertical is where x is near zero and y is near -1.

Looking at this post, I notice that if you want to convert this vector into angles, you can use the following algorithms.

If you want to calculate how many degrees from vertical the device is rotated (where positive is clockwise, negative is counter-clockwise), you can calculate this as:

// how much is it rotated around the z axis

CGFloat angle = atan2(y, x) + M_PI_2;           // in radians
CGFloat angleDegrees = angle * 180.0f / M_PI;   // in degrees
You can use this to figure out how much to rotate the view via the Quartz 2D transform property:

self.view.layer.transform = CATransform3DRotate(CATransform3DIdentity, -rotateRadians, 0, 0, 1);
(Personally, I update the rotation angle in the startDeviceMotionUpdates method, and update this transform in a CADisplayLink, which decouples the screen updates from the angle updates.)

You can see how far you've tilted it backward/forward via:

// how far it it tilted forward and backward

CGFloat r = sqrtf(x*x + y*y + z*z);
CGFloat tiltForwardBackward = acosf(z/r) * 180.0f / M_PI - 90.0f;

摘抄：
[measuring-tilt-angle-with-cmmotionmanager]

Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo][jekyll-gh].

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com
[measuring-tilt-angle-with-cmmotionmanager]:    http://stackoverflow.com/questions/15646433/measuring-tilt-angle-with-cmmotionmanager
